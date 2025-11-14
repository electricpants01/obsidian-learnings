# Widgets (WidgetKit)

## Setup

```swift
import WidgetKit
import SwiftUI

struct MyWidget: Widget {
    let kind = "MyWidget"
    
    var body: some WidgetConfiguration {
        StaticConfiguration(kind: kind, provider: Provider()) { entry in
            MyWidgetView(entry: entry)
        }
        .configurationDisplayName("My Widget")
        .description("Widget description")
    }
}
```

## Recursos
- [WidgetKit](https://developer.apple.com/documentation/widgetkit)
