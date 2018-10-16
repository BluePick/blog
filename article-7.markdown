# Explicit State in Swift

> [In information technology and computer science, a program is described as stateful if it is designed to remember preceding events or user interactions; the remembered information is called the state of the system.](https://en.wikipedia.org/wiki/State_(computer_science))

> [In computer science, functional programming is a programming paradigm—a style of building the structure and elements of computer programs—that treats computation as the evaluation of mathematical functions and **avoids changing-state** and mutable data.](https://en.wikipedia.org/wiki/Functional_programming)

State is unavoidable in applications with a user interface. A UI implicitly has state. Is the switch on or off? Does the text field contain text? Which view controller is at the top of the navigation stack? Whatever is rendered on-screen represents (partially) the state of the application.

A such, if our application is going to implicitly have state, we would be better off ***explicitly*** defining the possible states it can be in. This way — as software engineers — we can avoid bugs through making assumptions or mistakes about the state of the application at any given point.

## Example (iOS)

Let's say, we want a view that displays an image, which is loaded from a URL. Additionally, we want to display an activity indicator, for when the image is loading, as well as a text label to display an error message, or an empty message.

So, there are five possible states this view can be in.

1. The initial state, where nothing has yet happened.
2. The loading state, where we are waiting for the image to be downloaded.
3. The loaded state, where the image has been successfully downloaded and rendered.
4. The empty state, where the URL explicitly does not have an associated image.
5. The error state, where an error occurred in loading an image from the URL.

Our view needs to be able to configure and render itself for all of these possible states.

```swift
final class URLImageView: UIView {

    private let activityIndicatorView: UIActivityIndicatorView
    private let imageView: UIImageView
    private let textLabel: UILabel

    override init(frame: CGRect) {

        self.activityIndicatorView = UIActivityIndicatorView(frame: .zero)
        self.activityIndicatorView.translatesAutoresizingMaskIntoConstraints = false

        self.imageView = UIImageView(frame: .zero)
        self.imageView.translatesAutoresizingMaskIntoConstraints = false

        self.textLabel = UILabel(frame: .zero)
        self.textLabel.translatesAutoresizingMaskIntoConstraints = false

        super.init(frame: frame)

        self.addSubview(self.imageView)
        self.addSubview(self.textLabel)
        self.addSubview(self.activityIndicatorView)

        NSLayoutConstraint.activate([

            self.activityIndicatorView.centerXAnchor.constraint(equalTo: self.centerXAnchor),
            self.activityIndicatorView.centerYAnchor.constraint(equalTo: self.centerYAnchor),

            self.imageView.topAnchor.constraint(equalTo: self.topAnchor),
            self.imageView.bottomAnchor.constraint(equalTo: self.bottomAnchor),
            self.imageView.leadingAnchor.constraint(equalTo: self.leadingAnchor),
            self.imageView.trailingAnchor.constraint(equalTo: self.trailingAnchor),

            self.textLabel.topAnchor.constraint(equalTo: self.topAnchor),
            self.textLabel.bottomAnchor.constraint(equalTo: self.bottomAnchor),
            self.textLabel.leadingAnchor.constraint(equalTo: self.leadingAnchor),
            self.textLabel.trailingAnchor.constraint(equalTo: self.trailingAnchor),
            ])
    }

    required init?(coder _: NSCoder) {
        fatalError()
    }
}
```

Here we have a view with three subviews, an activity indicator, an image view and a text label. These subviews will need configuring for the five possible states it could be in.

```swift
final class URLImageView: UIView {

    enum State {
        case initial
        case loading
        case loaded(UIImage)
        case empty(String)
        case error(Error)
    }

    var state: State = .initial
}
```

Here we use an `enum`, with five cases, and three associated values. And, we add a `state` property to the view to allow the state to be explicitly set.

```swift
var state: State = .initial {
    didSet {
        switch self.state {
        case .initial:
            self.activityIndicatorView.stopAnimating()
            self.imageView.image = nil
            self.textLabel.text = nil
        case .loading:
            self.activityIndicatorView.startAnimating()
            self.imageView.image = nil
            self.textLabel.text = nil
        case .loaded(let image):
            self.activityIndicatorView.stopAnimating()
            self.imageView.image = image
            self.textLabel.text = nil
        case .empty(let message):
            self.activityIndicatorView.stopAnimating()
            self.imageView.image = nil
            self.textLabel.text = message
        case .error(let error):
            self.activityIndicatorView.stopAnimating()
            self.imageView.image = nil
            self.textLabel.text = "\(error)"
        }
    }
}
```

Here we have added a `didSet` function to our `state` property so when the state changes we can configure the view according to the state it should be in.

It is now not possible for our view to be in any other than the five states that we have explicitly enumerated. Through architecture, we have removed the possibility of stateful bugs existing in our view.

## Addendum

Though outside the scope of this article, it is worth noting that the logic of changing states should be removed from the view, and encapsulated in the `State` model.

```swift
var state: State = .initial {
    didSet {
        self.state.isAnimating ? self.activityIndicatorView.startAnimating() : self.activityIndicatorView.stopAnimating()
        self.imageView.image = self.state.image
        self.textLabel.text = self.state.text
    }
}

extension URLImageView.State {

    var isAnimating: Bool {
        switch self {
        case .initial, .loaded, .empty, .error:
            return false
        case .loading
            return true
        }
    }

    var text: String? {
        switch self {
        case .initial, .loading, .loaded:
            return nil
        case .empty(let message):
            return message
        case .error(let error):
            return "\(error)"
        }
    }

    var image: UIImage? {
        switch self {
        case .initial, .loading, .empty, .error:
            return nil
        case .loaded(let image):
            return image
        }
    }
}
```

<table>
<tr>
<td><img src="https://oliverrussellwhite.github.io/hero.png"></td>
<td>
<p>Hello! I'm Oliver. I'm a software engineer, and tutor.</p>
<p>I've been building apps for &#63743; platforms for ten years, and tutoring software developers in Swift and iOS for over a year.</p>
<p><a href="mailto:fortandlangley@gmail.com">I would ♥︎ to work with you.</a></p>
</td>
</tr>
</table>
