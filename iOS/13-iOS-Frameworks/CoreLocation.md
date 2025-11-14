# CoreLocation

## Setup

```swift
import CoreLocation

class LocationManager: NSObject, CLLocationManagerDelegate {
    let manager = CLLocationManager()
    
    override init() {
        super.init()
        manager.delegate = self
        manager.requestWhenInUseAuthorization()
    }
    
    func startTracking() {
        manager.startUpdatingLocation()
    }
    
    func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        guard let location = locations.last else { return }
        print("Lat: \(location.coordinate.latitude), Lon: \(location.coordinate.longitude)")
    }
}
```

## Recursos
- [CoreLocation](https://developer.apple.com/documentation/corelocation)
