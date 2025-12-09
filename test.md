## 1. Haptics with UIImpactFeedbackGenerator
 
`UIImpactFeedbackGenerator` is a UIKit utility that lets us trigger haptic feedback – small vibrations that match user interactions. It makes UI feel more responsive and physical, for example:

- A light tap when a button is pressed  
- A stronger bump when confirming an action  
- Different strengths for different UI elements  

This utility is part of the iOS Haptic Feedback system and uses the Taptic Engine on supported devices.


## 2. How to Implement

### Step 1: Import UIKit  
Add at the top of `ViewController.swift`
```swift
import UIKit
```




### Step 2: Create a Custom View for the Buttons

Create a HapticsView that is responsible for the layout and appearance of the buttons.

```swift
class HapticsView: UIView {

    let lightButton = UIButton(type: .system)
    let mediumButton = UIButton(type: .system)
    let heavyButton = UIButton(type: .system)

    override init(frame: CGRect) {
        super.init(frame: frame)
        backgroundColor = .systemBackground
        setupUI()
    }

    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    private func setupUI() {
        lightButton.setTitle("Light", for: .normal)
        mediumButton.setTitle("Medium", for: .normal)
        heavyButton.setTitle("Heavy", for: .normal)

        let stack = UIStackView(arrangedSubviews: [lightButton, mediumButton, heavyButton])
        stack.axis = .vertical
        stack.spacing = 16

        stack.translatesAutoresizingMaskIntoConstraints = false
        addSubview(stack)

        NSLayoutConstraint.activate([
            stack.centerXAnchor.constraint(equalTo: centerXAnchor),
            stack.centerYAnchor.constraint(equalTo: centerYAnchor)
        ])
    }
}

```

### Step 3: Use the Custom View in the View Controller

In the view controller, we use HapticsView as the main view and attach actions for each button.

```swift
class ViewController: UIViewController {

    override func loadView() {
        view = HapticsView()
    }

    override func viewDidLoad() {
        super.viewDidLoad()

        let hapticsView = view as! HapticsView

        hapticsView.lightButton.addTarget(self, action: #selector(lightImpact), for: .touchUpInside)
        hapticsView.mediumButton.addTarget(self, action: #selector(mediumImpact), for: .touchUpInside)
        hapticsView.heavyButton.addTarget(self, action: #selector(heavyImpact), for: .touchUpInside)
    }

    @objc func lightImpact() {
        let g = UIImpactFeedbackGenerator(style: .light)
        g.prepare()
        g.impactOccurred()
    }

    @objc func mediumImpact() {
        let g = UIImpactFeedbackGenerator(style: .medium)
        g.prepare()
        g.impactOccurred()
    }

    @objc func heavyImpact() {
        let g = UIImpactFeedbackGenerator(style: .heavy)
        g.prepare()
        g.impactOccurred()
    }
}

```

### Step 4: Reuse a Generator (Optional)

If you trigger haptics frequently (e.g., in a repeated interaction), you can store the generator as a property and reuse it:

```swift
class ViewController: UIViewController {

    private let mediumGenerator = UIImpactFeedbackGenerator(style: .medium)

    override func viewDidLoad() {
        super.viewDidLoad()

        let hapticsView = view as! HapticsView
        hapticsView.mediumButton.addTarget(self, action: #selector(triggerMedium), for: .touchUpInside)

        mediumGenerator.prepare()
    }

    @objc func triggerMedium() {
        mediumGenerator.impactOccurred()
    }
}
```

This avoids recreating the generator each time and can slightly improve responsiveness.


## 3. Documentation

### 3.1 `UIImpactFeedbackGenerator` Overview

**Purpose:**  
Produce physical “impact” sensations that match UI collisions or button presses.

**Main API:**

```swift
let generator = UIImpactFeedbackGenerator(style: .medium)
generator.prepare()
generator.impactOccurred()
```

- **Initializer**
  - `init(style: UIImpactFeedbackGenerator.FeedbackStyle)`
  - Styles:
    - `.light`
    - `.medium`
    - `.heavy`
    - `.soft`
    - `.rigid` 

- **Methods**
  - `prepare()`
    - Tells the system to prepare the Taptic Engine.
    - Call shortly before `impactOccurred()` (for example, when a touch begins).
  - `impactOccurred()`
    - Triggers the actual haptic impact.
  - `impactOccurred(intensity: CGFloat)`
    - Allows adjusting intensity between 0.0 and 1.0.

### 3.2 Usage Pattern and Best Practices

- Create a **new generator** or reuse one when appropriate.
- Call `prepare()` just before the user action (e.g., `touchDown`) to reduce latency.
- Trigger `impactOccurred()` when the action is confirmed (e.g., on button tap).
- Haptics only work on devices that support them (e.g., iPhones with Taptic Engine). On unsupported devices, calling these methods is safe; nothing happens.
- Do not overuse haptics; use them for **key interactions**:
  - Confirm or reject actions
  - Changing selections in a picker
  - Reaching the end of a scroll

### 3.3 Related Haptic Generators

Besides `UIImpactFeedbackGenerator`, UIKit also offers:

- `UINotificationFeedbackGenerator`
  - For **success**, **warning**, **error** patterns.
- `UISelectionFeedbackGenerator`
  - For small movements between discrete values (picker wheels, segmented controls).



## 4. Reference Code 
The reference code below demonstrates a full programmatic implementation.
```swift
import UIKit

// MARK: Sample View
class HapticsView: UIView {

    let lightButton = UIButton(type: .system)
    let mediumButton = UIButton(type: .system)
    let heavyButton = UIButton(type: .system)

    override init(frame: CGRect) {
        super.init(frame: frame)
        backgroundColor = .systemBackground
        setupButtons()
    }

    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    private func setupButtons() {
        lightButton.setTitle("Light Impact", for: .normal)
        mediumButton.setTitle("Medium Impact", for: .normal)
        heavyButton.setTitle("Heavy Impact", for: .normal)

        [lightButton, mediumButton, heavyButton].forEach {
            $0.titleLabel?.font = .systemFont(ofSize: 22)
        }

        let stack = UIStackView(arrangedSubviews: [lightButton, mediumButton, heavyButton])
        stack.axis = .vertical
        stack.spacing = 20
        stack.alignment = .center

        stack.translatesAutoresizingMaskIntoConstraints = false
        addSubview(stack)

        NSLayoutConstraint.activate([
            stack.centerXAnchor.constraint(equalTo: centerXAnchor),
            stack.centerYAnchor.constraint(equalTo: centerYAnchor)
        ])
    }
}


// MARK: ViewController
class ViewController: UIViewController {

    override func loadView() {
        view = HapticsView()
    }

    override func viewDidLoad() {
        super.viewDidLoad()

        let hapticsView = view as! HapticsView

        hapticsView.lightButton.addTarget(self, action: #selector(lightImpactTapped), for: .touchUpInside)
        hapticsView.mediumButton.addTarget(self, action: #selector(mediumImpactTapped), for: .touchUpInside)
        hapticsView.heavyButton.addTarget(self, action: #selector(heavyImpactTapped), for: .touchUpInside)
    }

    @objc func lightImpactTapped() {
        let generator = UIImpactFeedbackGenerator(style: .light)
        generator.prepare()
        generator.impactOccurred()
    }

    @objc func mediumImpactTapped() {
        let generator = UIImpactFeedbackGenerator(style: .medium)
        generator.prepare()
        generator.impactOccurred()
    }

    @objc func heavyImpactTapped() {
        let generator = UIImpactFeedbackGenerator(style: .heavy)
        generator.prepare()
        generator.impactOccurred()
    }
}
```
