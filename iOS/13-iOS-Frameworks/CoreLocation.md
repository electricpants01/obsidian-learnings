# CoreLocation - Localización en iOS

## Setup

```swift
import CoreLocation

class LocationManager: NSObject, CLLocationManagerDelegate {
    let manager = CLLocationManager()
    var locationHandler: ((CLLocation) -> Void)?
    
    override init() {
        super.init()
        manager.delegate = self
        manager.desiredAccuracy = kCLLocationAccuracyBest
    }
    
    func requestPermission() {
        manager.requestWhenInUseAuthorization()
        // o
        // manager.requestAlwaysAuthorization()
    }
    
    func startTracking() {
        manager.startUpdatingLocation()
    }
    
    func stopTracking() {
        manager.stopUpdatingLocation()
    }
    
    // Delegate
    func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        guard let location = locations.last else { return }
        locationHandler?(location)
    }
    
    func locationManager(_ manager: CLLocationManager, didFailWithError error: Error) {
        print("Location error: \(error)")
    }
}
```

## SwiftUI Integration

```swift
@MainActor
class LocationViewModel: NSObject, ObservableObject, CLLocationManagerDelegate {
    @Published var location: CLLocation?
    @Published var authorizationStatus: CLAuthorizationStatus = .notDetermined
    
    private let manager = CLLocationManager()
    
    override init() {
        super.init()
        manager.delegate = self
        manager.desiredAccuracy = kCLLocationAccuracyBest
    }
    
    func requestPermission() {
        manager.requestWhenInUseAuthorization()
    }
    
    func startTracking() {
        manager.startUpdatingLocation()
    }
    
    func locationManagerDidChangeAuthorization(_ manager: CLLocationManager) {
        authorizationStatus = manager.authorizationStatus
    }
    
    func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        location = locations.last
    }
}

struct LocationView: View {
    @StateObject private var viewModel = LocationViewModel()
    
    var body: some View {
        VStack {
            if let location = viewModel.location {
                Text("Lat: \(location.coordinate.latitude)")
                Text("Lon: \(location.coordinate.longitude)")
            }
            
            Button("Request Location") {
                viewModel.requestPermission()
                viewModel.startTracking()
            }
        }
    }
}
```

## Geocoding

```swift
func reverseGeocode(location: CLLocation) async throws -> String {
    let geocoder = CLGeocoder()
    let placemarks = try await geocoder.reverseGeocodeLocation(location)
    
    guard let placemark = placemarks.first else {
        throw LocationError.noPlacemark
    }
    
    return placemark.locality ?? "Unknown"
}

// Forward geocoding
func geocodeAddress(_ address: String) async throws -> CLLocation {
    let geocoder = CLGeocoder()
    let placemarks = try await geocoder.geocodeAddressString(address)
    
    guard let location = placemarks.first?.location else {
        throw LocationError.noLocation
    }
    
    return location
}
```

## Info.plist

```xml
<key>NSLocationWhenInUseUsageDescription</key>
<string>We need your location to show nearby places</string>

<key>NSLocationAlwaysUsageDescription</key>
<string>We need your location for background tracking</string>
```

## Recursos

- 📚 [CoreLocation](https://developer.apple.com/documentation/corelocation)
- 🎥 [Location Services](https://www.youtube.com/watch?v=1b4BhPWNw0o)
