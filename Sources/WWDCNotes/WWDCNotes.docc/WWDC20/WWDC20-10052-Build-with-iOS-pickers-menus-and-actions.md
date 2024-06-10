# Build with iOS pickers, menus and actions

Build iPhone and iPad apps with fluid interfaces and easily-accessible contextual information. We’ll show you how to integrate the latest UIKit controls into your app to best take advantage of menus, date pickers, page controls, and segmented controllers. Learn how to adopt Menus throughly your user interface, and explore how UIAction can help unify your event handling.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10052", purpose: link, label: "Watch Video (20 min)")

   @Contributors {
      @GitHubUser(ATahhan)
   }
}



## `UIControl`s New Design and Appearance

* [`UISlider`][sliderDoc] and [`UIProgressView`][progressDoc] have now more consistent UI across all platforms
* [`UIActivityIndicatorView`][activityDoc] and [`UIPickerView`][pickerDoc] have new designs as well

## [`UIPageControl`][pageDoc]

* `UIPageControl` can now be scrubbed and scrolled when the number of pages doesn’t fit the available space. 
* `UIPageControl` components such as indicators can now be customized via the following API:

```swift
let pageControl = UIPageControl()
pageControl.numberOfPages = 5

pageControl.backgroundStyle = .prominent

pageControl.preferredIndicatorImage =
    UIImage(systemName: "bookmark.fill")

pageControl.setIndicatorImage(
    UIImage(systemName: "heart.fill"), forPage: 0)
```

![][UIPageControl]

## [`UIColorPickerViewController`][colorPickerDoc]

* New `UIColorPickerViewController`, presented as a sheet and featuring eyedropper, favorites, and hexadecimal specification

```swift
var color = UIColor.blue
var colorPicker = UIColorPickerViewController()

func pickColor() {
    colorPicker.supportsAlpha = true
    colorPicker.selectedColor = color
    self.present(
        colorPicker,
        animated: true,
        completion: nil
    )
}

func colorPickerViewControllerDidSelectColor(
    _ viewController: UIColorPickerViewController
) {
    color = viewController.selectedColor
}

func colorPickerViewControllerDidFinish(
    _ viewController: UIColorPickerViewController
) {
    // Do nothing
}
```

![][UIColorPickerController]

## [`UIDatePicker`][datePickerDoc]

* `UIDatePicker` now offers a compact style, and we can optionally limit selection to just a date or time.

```swift
let datePicker = UIDatePicker()
datePicker.date = Date(
    timeIntervalSinceReferenceDate: timeInterval
)

datePicker.preferredDatePickerStyle = .compact

datePicker.calendar = Calendar(identifier: .japanese)
datePicker.datePickerMode = .date

datePicker.addTarget(
    self,
    action: #selector(dateSet),
    for: .valueChanged
)
```

| ![][UIDatePickeriOS] | ![][UIDatePickermacOS] |
| ----------- | ----------- |
| iOS / iPadOS | macOS |

## [`UIMenu`][menuDoc]

* In iOS, we can now add `UIMenu`s to `UIButtons` and `UIBarButtonItem`s displayed to the user on a long press:

```swift
button.menu = UIMenu(...)

barButtonItem.menu = UIMenu(...)
```

* We can also adjust the behavior of menus to show immediately on touch down instead of waiting for a long press by:
  * `button.showsMenuAsPrimaryAction = true` for `UIButton`
  * Not providing a primary action for `UIBarButtonItem`

* `UINavigationBar` back button now gets a default menu as well (to let the user jumps back to a previous controller quickly). Menu titles are automatically chosen, if we’re currently using a custom title view in our controller, consider setting:
  * `.backBarButtonItem.title`
  * `.backButtonTitle`
  * `.title`

![][NavigationBarBackButtonMenu]

* We can access `UIControl` interactions and their properties, and we can subclass `UIControl` and provide a custom implementation for our custom menu based UIs
	 
* We can use [`UIDeferredMenuElement`][deferredDoc] to add the ability to load menu items asynchronously while showing a standard loading UI.

![][UIDeferredMenuElement] 

## [`UIAction`][actiondoc]

* `UIAction` can now be directly passed to `UIButton`, `UIBarButtonItem`, and `UISegmentedControl` initializers.
* `UIButton` initializer `init(type:primaryAction:)` defaults the `UIButton` type to `.system`, which sets the button’s title and image to that of the primary action (`primaryAction.title` and `primaryAction.image`)
* We can now create `UISegmentedControl` as easy as:

```swift
let control = UISegmentedControl(frame: .zero, actions: Colors.allCases.map { color in 
	UIAction(title: color.description) { [unowned imageView] _ in 
		imageView.tintColor = color.color()
	}
})
```

[sliderDoc]: https://developer.apple.com/documentation/uikit/uislider
[progressDoc]: https://developer.apple.com/documentation/uikit/UIProgressView
[activityDoc]: https://developer.apple.com/documentation/uikit/UIActivityIndicatorView
[pickerDoc]: https://developer.apple.com/documentation/uikit/UIPickerView
[pageDoc]: https://developer.apple.com/documentation/uikit/UIPageControl
[colorPickerDoc]: https://developer.apple.com/documentation/uikit/uicolorpickerviewcontroller
[datePickerDoc]: https://developer.apple.com/documentation/uikit/UIDatePicker
[menuDoc]: https://developer.apple.com/documentation/uikit/UIMenu
[deferredDoc]: https://developer.apple.com/documentation/uikit/UIDeferredMenuElement
[actionDoc]: https://developer.apple.com/documentation/uikit/uiaction

[UIPageControl]: WWDC20-10052-UIPageControl
[UIColorPickerController]: WWDC20-10052-UIColorPickerController
[UIDatePickeriOS]: WWDC20-10052-UIDatePicker-iOS
[UIDatePickermacOS]: WWDC20-10052-UIDatePicker-macOS
[NavigationBarBackButtonMenu]: WWDC20-10052-NavigationBarBackButtonMenu
[UIDeferredMenuElement]: WWDC20-10052-UIDeferredMenuElement
