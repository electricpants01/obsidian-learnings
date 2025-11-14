# View Controllers en UIKit

## UIViewController Lifecycle

```swift
class MyViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        // View cargada en memoria
        // Setup inicial, constraints, etc.
    }
    
    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        // Vista a punto de aparecer
        // Refresh data, start animations
    }
    
    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        // Vista visible en pantalla
        // Start heavy tasks, analytics
    }
    
    override func viewWillDisappear(_ animated: Bool) {
        super.viewWillDisappear(animated)
        // Vista a punto de desaparecer
        // Save data, stop animations
    }
    
    override func viewDidDisappear(_ animated: Bool) {
        super.viewDidDisappear(animated)
        // Vista ya no visible
        // Cleanup resources
    }
}
```

## Programmatic UI

```swift
class CustomViewController: UIViewController {
    private let titleLabel: UILabel = {
        let label = UILabel()
        label.text = "Title"
        label.font = .systemFont(ofSize: 24, weight: .bold)
        label.translatesAutoresizingMaskIntoConstraints = false
        return label
    }()
    
    private let button: UIButton = {
        let button = UIButton(type: .system)
        button.setTitle("Tap Me", for: .normal)
        button.translatesAutoresizingMaskIntoConstraints = false
        return button
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
    }
    
    private func setupUI() {
        view.backgroundColor = .white
        
        view.addSubview(titleLabel)
        view.addSubview(button)
        
        NSLayoutConstraint.activate([
            titleLabel.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            titleLabel.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 20),
            
            button.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            button.centerYAnchor.constraint(equalTo: view.centerYAnchor)
        ])
        
        button.addTarget(self, action: #selector(buttonTapped), for: .touchUpInside)
    }
    
    @objc private func buttonTapped() {
        print("Button tapped")
    }
}
```

## Navigation

```swift
// Push
let detailVC = DetailViewController()
navigationController?.pushViewController(detailVC, animated: true)

// Pop
navigationController?.popViewController(animated: true)

// Pop to root
navigationController?.popToRootViewController(animated: true)

// Present modally
let modalVC = ModalViewController()
present(modalVC, animated: true)

// Dismiss
dismiss(animated: true)
```

## Recursos

- 📚 [UIViewController](https://developer.apple.com/documentation/uikit/uiviewcontroller)
- 🎥 [UIKit Tutorial](https://www.youtube.com/watch?v=pL7KMU1U3_E)
