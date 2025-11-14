# Fastlane

## Instalación

```bash
sudo gem install fastlane -NV
```

## Fastfile

```ruby
default_platform(:ios)

platform :ios do
  desc "Build and test"
  lane :test do
    scan
  end
  
  desc "Deploy to TestFlight"
  lane :beta do
    increment_build_number
    build_app(scheme: "MyApp")
    upload_to_testflight
  end
end
```

## Recursos
- [Fastlane](https://fastlane.tools)
