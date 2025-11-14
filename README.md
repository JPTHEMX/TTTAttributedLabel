Code Cleanup
Dead Code and Legacy Code

The ViewModel contains dead code and legacy code that is not being used in the current application flow, unnecessarily increasing the binary size and hindering maintainability.

Unnecessary Initialization

The ViewModel forces initialization of properties and dependencies that are not used, violating the Single Responsibility Principle (SRP) and generating unnecessary memory overhead.
Initializers require parameters that are not consumed, increasing the complexity of dependency injection and hindering testability.

Unused Parameters

Methods exist with unused parameters, which generates compiler warnings and reduces the clarity of the function contract.

Single Responsibility Principle (SRP) Violation

Some classes have multiple responsibilities (God Objects), violating SOLID principles. This results in:

Low cohesion
High coupling
Difficulty in unit testing
Code that is hard to maintain and extend



Inconsistency in ViewController Hierarchy

BaseViewController is not being used consistently across the application. For example, SearchViewController and other VCs don't inherit from BaseViewController, causing:

Duplication of common code (UI setup, lifecycle methods, etc.)
Inconsistency in UI behavior
Loss of shared functionality
Difficulty applying global changes to all ViewControllers



Code Duplication (DRY Violation)

Duplicated code has been identified, violating the DRY (Don't Repeat Yourself) principle, which generates:

Inconsistencies when making changes
Larger bug surface area
Difficulty refactoring
