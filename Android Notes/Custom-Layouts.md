# Custom Layouts en Jetpack Compose

## Layout Básico con Layout Modifier

### 1. Layout Modifier Simple

```kotlin
@Composable
fun SimpleCustomLayout() {
    Box(
        Modifier
            .fillMaxSize()
            .layout { measurable, constraints ->
                val placeable = measurable.measure(constraints)
                
                // Definir el tamaño del layout
                layout(placeable.width, placeable.height) {
                    // Posicionar el elemento
                    placeable.place(0, 0)
                }
            }
    )
}
```

### 2. Layout con Offset Personalizado

```kotlin
fun Modifier.customOffset(x: Dp, y: Dp) = layout { measurable, constraints ->
    val placeable = measurable.measure(constraints)
    
    layout(placeable.width, placeable.height) {
        placeable.place(
            x.roundToPx(),
            y.roundToPx()
        )
    }
}

// Uso
@Composable
fun OffsetExample() {
    Text(
        "Custom Offset",
        Modifier.customOffset(x = 50.dp, y = 100.dp)
    )
}
```

## Layout Composable Personalizado

### 1. Custom Row

```kotlin
@Composable
fun CustomRow(
    modifier: Modifier = Modifier,
    content: @Composable () -> Unit
) {
    Layout(
        modifier = modifier,
        content = content
    ) { measurables, constraints ->
        // Medir todos los children
        val placeables = measurables.map { measurable ->
            measurable.measure(constraints.copy(minWidth = 0))
        }
        
        // Calcular tamaño total
        val width = placeables.sumOf { it.width }
        val height = placeables.maxOf { it.height }
        
        // Definir el layout
        layout(width, height) {
            var xPosition = 0
            
            // Posicionar cada child
            placeables.forEach { placeable ->
                placeable.place(x = xPosition, y = 0)
                xPosition += placeable.width
            }
        }
    }
}
```

### 2. Custom Column con Spacing

```kotlin
@Composable
fun CustomColumn(
    modifier: Modifier = Modifier,
    spacing: Dp = 0.dp,
    content: @Composable () -> Unit
) {
    Layout(
        modifier = modifier,
        content = content
    ) { measurables, constraints ->
        val spacingPx = spacing.roundToPx()
        
        val placeables = measurables.map { measurable ->
            measurable.measure(constraints.copy(minHeight = 0))
        }
        
        val width = placeables.maxOf { it.width }
        val height = placeables.sumOf { it.height } + 
                    (spacingPx * (placeables.size - 1).coerceAtLeast(0))
        
        layout(width, height) {
            var yPosition = 0
            
            placeables.forEach { placeable ->
                placeable.place(x = 0, y = yPosition)
                yPosition += placeable.height + spacingPx
            }
        }
    }
}
```

## SubcomposeLayout - Layouts Dinámicos

### 1. Measured Height Layout

```kotlin
@Composable
fun MeasuredHeightLayout(
    viewToMeasure: @Composable () -> Unit,
    dependentView: @Composable (IntSize) -> Unit
) {
    SubcomposeLayout { constraints ->
        // Primero medir viewToMeasure
        val measuredPlaceable = subcompose("measured", viewToMeasure)
            .map { it.measure(constraints) }
        
        val measuredSize = IntSize(
            width = measuredPlaceable.maxOf { it.width },
            height = measuredPlaceable.maxOf { it.height }
        )
        
        // Luego componer dependentView con el tamaño medido
        val dependentPlaceables = subcompose("dependent") {
            dependentView(measuredSize)
        }.map { it.measure(constraints) }
        
        val width = maxOf(
            measuredPlaceable.maxOf { it.width },
            dependentPlaceables.maxOf { it.width }
        )
        val height = measuredPlaceable.sumOf { it.height } +
                    dependentPlaceables.sumOf { it.height }
        
        layout(width, height) {
            var yPosition = 0
            
            measuredPlaceable.forEach { placeable ->
                placeable.place(0, yPosition)
                yPosition += placeable.height
            }
            
            dependentPlaceables.forEach { placeable ->
                placeable.place(0, yPosition)
                yPosition += placeable.height
            }
        }
    }
}

// Uso
@Composable
fun ExampleUsage() {
    MeasuredHeightLayout(
        viewToMeasure = {
            Text("Header")
        },
        dependentView = { size ->
            Text("Content that knows header size: ${size.height}px")
        }
    )
}
```

### 2. Lazy Grid Staggered

```kotlin
@Composable
fun StaggeredGrid(
    columns: Int,
    modifier: Modifier = Modifier,
    content: @Composable () -> Unit
) {
    Layout(
        modifier = modifier,
        content = content
    ) { measurables, constraints ->
        val columnWidth = constraints.maxWidth / columns
        val columnConstraints = constraints.copy(
            minWidth = 0,
            maxWidth = columnWidth
        )
        
        val placeables = measurables.map { measurable ->
            measurable.measure(columnConstraints)
        }
        
        val columnHeights = IntArray(columns) { 0 }
        
        val positions = placeables.map { placeable ->
            val column = columnHeights.indices.minByOrNull { columnHeights[it] } ?: 0
            val x = column * columnWidth
            val y = columnHeights[column]
            
            columnHeights[column] += placeable.height
            
            IntOffset(x, y)
        }
        
        val height = columnHeights.maxOrNull() ?: 0
        
        layout(constraints.maxWidth, height) {
            placeables.forEachIndexed { index, placeable ->
                placeable.place(positions[index])
            }
        }
    }
}
```

## Intrinsic Measurements

### 1. IntrinsicSize

```kotlin
@Composable
fun IntrinsicSizeExample() {
    Row(
        modifier = Modifier
            .fillMaxWidth()
            .height(IntrinsicSize.Min)
    ) {
        Text(
            "Short",
            Modifier
                .fillMaxHeight()
                .background(Color.Red)
        )
        Divider(
            color = Color.Black,
            modifier = Modifier
                .fillMaxHeight()
                .width(1.dp)
        )
        Text(
            "Very very very long text",
            Modifier
                .fillMaxHeight()
                .background(Color.Blue)
        )
    }
}
```

### 2. Custom Layout con Intrinsics

```kotlin
@Composable
fun CustomIntrinsicLayout(
    modifier: Modifier = Modifier,
    content: @Composable () -> Unit
) {
    Layout(
        modifier = modifier,
        content = content,
        measurePolicy = object : MeasurePolicy {
            override fun MeasureScope.measure(
                measurables: List<Measurable>,
                constraints: Constraints
            ): MeasureResult {
                val placeables = measurables.map { it.measure(constraints) }
                val width = placeables.maxOf { it.width }
                val height = placeables.sumOf { it.height }
                
                return layout(width, height) {
                    var y = 0
                    placeables.forEach { placeable ->
                        placeable.place(0, y)
                        y += placeable.height
                    }
                }
            }
            
            override fun IntrinsicMeasureScope.minIntrinsicHeight(
                measurables: List<IntrinsicMeasurable>,
                width: Int
            ): Int {
                return measurables.sumOf { it.minIntrinsicHeight(width) }
            }
            
            override fun IntrinsicMeasureScope.maxIntrinsicHeight(
                measurables: List<IntrinsicMeasurable>,
                width: Int
            ): Int {
                return measurables.sumOf { it.maxIntrinsicHeight(width) }
            }
        }
    )
}
```

## Layouts Complejos

### 1. Masonry Layout

```kotlin
@Composable
fun MasonryLayout(
    columns: Int = 2,
    modifier: Modifier = Modifier,
    content: @Composable () -> Unit
) {
    Layout(
        modifier = modifier,
        content = content
    ) { measurables, constraints ->
        val columnWidth = constraints.maxWidth / columns
        val itemConstraints = constraints.copy(
            minWidth = 0,
            maxWidth = columnWidth
        )
        
        val placeables = measurables.map { measurable ->
            measurable.measure(itemConstraints)
        }
        
        val columnHeights = MutableList(columns) { 0 }
        val itemPositions = mutableListOf<IntOffset>()
        
        placeables.forEach { placeable ->
            val shortestColumn = columnHeights.withIndex()
                .minByOrNull { it.value }!!.index
            
            val x = shortestColumn * columnWidth
            val y = columnHeights[shortestColumn]
            
            itemPositions.add(IntOffset(x, y))
            columnHeights[shortestColumn] += placeable.height
        }
        
        val height = columnHeights.maxOrNull() ?: 0
        
        layout(constraints.maxWidth, height) {
            placeables.forEachIndexed { index, placeable ->
                placeable.place(itemPositions[index])
            }
        }
    }
}
```

### 2. Diagonal Layout

```kotlin
@Composable
fun DiagonalLayout(
    modifier: Modifier = Modifier,
    content: @Composable () -> Unit
) {
    Layout(
        modifier = modifier,
        content = content
    ) { measurables, constraints ->
        val placeables = measurables.map { it.measure(constraints) }
        
        var width = 0
        var height = 0
        
        placeables.forEach { placeable ->
            width += placeable.width
            height += placeable.height
        }
        
        layout(width, height) {
            var x = 0
            var y = 0
            
            placeables.forEach { placeable ->
                placeable.place(x, y)
                x += placeable.width
                y += placeable.height
            }
        }
    }
}
```

### 3. Circular Layout

```kotlin
@Composable
fun CircularLayout(
    radius: Dp = 100.dp,
    modifier: Modifier = Modifier,
    content: @Composable () -> Unit
) {
    Layout(
        modifier = modifier,
        content = content
    ) { measurables, constraints ->
        val radiusPx = radius.toPx()
        val placeables = measurables.map { it.measure(constraints) }
        
        val layoutSize = (radiusPx * 2).toInt()
        
        layout(layoutSize, layoutSize) {
            val angleStep = 360f / placeables.size
            
            placeables.forEachIndexed { index, placeable ->
                val angle = Math.toRadians((angleStep * index).toDouble())
                val x = (radiusPx + radiusPx * cos(angle) - placeable.width / 2).toInt()
                val y = (radiusPx + radiusPx * sin(angle) - placeable.height / 2).toInt()
                
                placeable.place(x, y)
            }
        }
    }
}
```

## ParentDataModifier

```kotlin
data class LayoutInfo(val weight: Float)

val Measurable.layoutWeight: Float
    get() = (parentData as? LayoutInfo)?.weight ?: 1f

fun Modifier.layoutWeight(weight: Float) = this.then(
    object : ParentDataModifier {
        override fun Density.modifyParentData(parentData: Any?) = 
            LayoutInfo(weight)
    }
)

@Composable
fun WeightedRow(
    modifier: Modifier = Modifier,
    content: @Composable () -> Unit
) {
    Layout(
        modifier = modifier,
        content = content
    ) { measurables, constraints ->
        val totalWeight = measurables.sumOf { it.layoutWeight.toDouble() }.toFloat()
        
        val placeables = measurables.map { measurable ->
            val itemWeight = measurable.layoutWeight
            val itemWidth = (constraints.maxWidth * (itemWeight / totalWeight)).toInt()
            
            measurable.measure(
                constraints.copy(
                    minWidth = 0,
                    maxWidth = itemWidth
                )
            )
        }
        
        val width = placeables.sumOf { it.width }
        val height = placeables.maxOf { it.height }
        
        layout(width, height) {
            var x = 0
            placeables.forEach { placeable ->
                placeable.place(x, 0)
                x += placeable.width
            }
        }
    }
}

// Uso
@Composable
fun WeightedRowExample() {
    WeightedRow {
        Box(Modifier.layoutWeight(1f).background(Color.Red).height(50.dp))
        Box(Modifier.layoutWeight(2f).background(Color.Blue).height(50.dp))
        Box(Modifier.layoutWeight(1f).background(Color.Green).height(50.dp))
    }
}
```

## Lookahead Layout (Experimental)

```kotlin
@OptIn(ExperimentalComposeUiApi::class)
@Composable
fun LookaheadExample() {
    var expanded by remember { mutableStateOf(false) }
    
    LookaheadScope {
        Column(
            Modifier
                .fillMaxWidth()
                .clickable { expanded = !expanded }
        ) {
            Box(
                Modifier
                    .animateBounds()
                    .fillMaxWidth()
                    .height(if (expanded) 200.dp else 100.dp)
                    .background(Color.Blue)
            )
        }
    }
}

@OptIn(ExperimentalComposeUiApi::class)
fun Modifier.animateBounds() = this.then(
    object : LayoutModifier {
        override fun MeasureScope.measure(
            measurable: Measurable,
            constraints: Constraints
        ): MeasureResult {
            val placeable = measurable.measure(constraints)
            return layout(placeable.width, placeable.height) {
                placeable.place(0, 0)
            }
        }
    }
)
```

## Performance Tips

```kotlin
// ✅ BUENO - Reusar objetos
@Composable
fun EfficientLayout(
    modifier: Modifier = Modifier,
    content: @Composable () -> Unit
) {
    val constraints = remember { Constraints() }
    
    Layout(
        modifier = modifier,
        content = content
    ) { measurables, incomingConstraints ->
        // ... implementación
        layout(0, 0) { }
    }
}

// ✅ BUENO - Evitar allocaciones innecesarias
@Composable
fun OptimizedLayout() {
    Layout(
        content = { }
    ) { measurables, constraints ->
        val placeables = measurables.map { 
            it.measure(constraints) 
        }
        
        // Evitar crear listas nuevas
        val width = placeables.maxOfOrNull { it.width } ?: 0
        val height = placeables.maxOfOrNull { it.height } ?: 0
        
        layout(width, height) {
            placeables.forEach { it.place(0, 0) }
        }
    }
}
```

## Recursos

- [Custom Layouts](https://developer.android.com/jetpack/compose/layouts/custom)
- [Layout Basics](https://developer.android.com/jetpack/compose/layouts/basics)
- [Intrinsic Measurements](https://developer.android.com/jetpack/compose/layouts/intrinsic-measurements)
- [SubcomposeLayout](https://developer.android.com/reference/kotlin/androidx/compose/ui/layout/SubcomposeLayout)

---
*Los custom layouts permiten crear UIs únicas y optimizadas*
