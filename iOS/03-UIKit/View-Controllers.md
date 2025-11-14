# View Controllers

## UIViewController Lifecycle

```swift
class MyViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        // View cargada en memoria
    }
    
    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        // Vista a punto de aparecer
    }
    
    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        // Vista visible
    }
    
    override func viewWillDisappear(_ animated: Bool) {
        super.viewWillDisappear(animated)
        // Vista a punto de desaparecer
    }
}
```

## Recursos
- [UIKit](https://developer.apple.com/documentation/uikit)
