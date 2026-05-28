# Eulerity Dynamic Campaign Form Engine

A production-ready, highly resilient iOS application built with SwiftUI following clean architecture principles. This application dynamically parses a backend JSON configuration payload, enforces visual theme overrides, sorts layout priorities, and securely manages a state-driven form validation lifecycle.

---

## 1. Overall Approach and Architecture

The project is structured entirely using the **MVVM (Model-View-ViewModel)** architectural pattern. This ensures complete separation of concerns between our data ingestion layer and the UI representation.

* **Models (`FormModels.swift`):** Standard Swift primitives matching the backend schema. It uses a custom container wrapper (`JSONValue`) to decode polymorphic JSON data payloads safely (handling default fields that can alternate between Strings, Integers, and Booleans) without triggering system-wide parsing failures.
* **ViewModel (`FormViewModel.swift`):** Serves as the central state engine for the interface. It maintains loading, error, and response capture states via `@Published` properties. The view model acts as a data pipeline, dynamically sorting elements and organizing user input inside a flat dictionary (`[String: Any]`) keyed directly by field IDs.
* **Views (`ContentView.swift` & Components):** Built using declarative SwiftUI. `ContentView` observes the view model and passes slice bindings down to a generic rendering engine (`FormFieldRouter`). The router translates field metadata configurations instantly into native structural elements like `TextField`, `Menu`, or `ColorPicker`.

---

## 2. Product Decisions & Edge-Case Guardrails

During implementation, several undefined product behaviors and data edge cases were resolved defensively to keep the experience bulletproof:

* **Asynchronous Content Sorting:** The array items in the incoming payload arrive completely out of sequential display order. To protect the user onboarding flow, a product decision was made to abstract the array through a computational layout property (`sortedFields`) that actively sorts elements ascendingly by their `order` integer before rendering.
* **Defensive Parsing Framework:** Real-world backends often ship unsupported layout mutations (such as an unannounced `DATE_PICKER` type). The decoding layer was extended with a fallback `.unknown` case. Instead of crashing the execution thread on an unexpected property string, the application ignores unsupported entities silently via `EmptyView` while rendering valid adjacent components correctly.
* **Validation Timing UX:** Validation checks are batched and executed specifically when the user taps "Submit Campaign" rather than checking inputs inline on every character keystroke. This prevents high-flicker validation warnings from showing while a user is mid-sentence, ensuring a polished, non-intrusive typing workflow.

---

## 3. What I Got Stuck On & Resolutions

* **The Cyrillic String Trap:** While reviewing the field enum structures, the text component type `"СНЕСКВОХ"` initially caused a silent failure in standard string mapping. On close inspection, the string contained hidden non-standard Cyrillic layout characters rather than standard English Unicode. 
    * *Resolution:* I injected a normalization sanitizing phase directly inside the `FieldType` custom enum initializer. This explicitly traps the Cyrillic sequence variance and normalizes it to a safe standard enum case, keeping the application structurally stable.
* **JSONSerialization Structural Crash:** When exporting the finalized results dictionary, `JSONSerialization` triggered an immediate native crash thread. This happened because the dictionary captures native SwiftUI `Color` objects from the user picker, which the default JSON serialization utility cannot interpret.
    * *Resolution:* I designed a dictionary data-cleansing pipeline. Before passing the collected keys to the encoder, the collection is iterated through. Any active SwiftUI UI tokens are parsed down to flat hex string components (`#HEX`), formatting a clean payload that passes validation seamlessly.

---

## 4. Future Enhancements (With More Time)

If granted additional development iterations, the following modules would be integrated:
1.  **Deep-Linked Metadata Execution:** Extend the metadata parsing dictionary inside legal checkbox modules to map string URLs into active, interactive web view components using `Link` modifiers.
2.  **Robust Unit Test Suites:** Author specialized XCTest cases verifying both the dynamic rendering sort priority matrix and the string encoding sanitizer.
3.  **Draft Auto-Persistence Integration:** Wire the state tracking layout dictionary directly into `SwiftData` or local sandbox caching, allowing campaign building tasks to resume work following accidental application kills.
