# Actors

## Actor Definición

```swift
actor BankAccount {
    private var balance: Double = 0
    
    func deposit(amount: Double) {
        balance += amount
    }
    
    func withdraw(amount: Double) -> Bool {
        guard balance >= amount else { return false }
        balance -= amount
        return true
    }
    
    func getBalance() -> Double {
        balance
    }
}
```

## Uso

```swift
let account = BankAccount()

Task {
    await account.deposit(amount: 100)
    let balance = await account.getBalance()
    print("Balance: \(balance)")
}
```

## Recursos
- [Actors](https://docs.swift.org/swift-book/LanguageGuide/Concurrency.html)
