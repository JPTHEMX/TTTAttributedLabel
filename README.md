1 - Business Results
In 2025, my focus has been on delivery velocity, architectural modernization, and proactive tech debt reduction, aligning with the "Promote code improvements" goal.

High Performance and Efficiency: I have consistently exceeded the team average by managing a volume of 20+ points per sprint. I regularly take on additional work (3-5 tickets) mid-sprint to respond to critical business needs, completing 100% of committed work with no carryover.

Time-to-Market Acceleration: I demonstrated radical refactoring efficiency on the Carousel Hub. I corrected code quality issues and errors from a previous implementation, delivering a robust and stable solution in just 2-3 weeks and drastically improving maintainability.

Architectural Modernization (ListView, Hub, Thematic, Wallet): I led the refactoring of key components (ListView, Hub, Thematic, Wallet), migrating from UICollectionViewFlowLayout to UICollectionViewCompositionalLayout and implementing UICollectionViewDiffableDataSource. This eliminated obsolete and error-prone code, establishing a clean and scalable codebase that reduces long-term maintenance costs.

Architectural Modernization (Global): I executed a strategic refactor of the DataManager across all our main components (Entry, Hub, Wallet, Shopping Hub, Activation Page, etc.). I centralized error-handling logic, removed redundant code, and began adopting async/await and Strict Concurrency, improving the stability and security of our data layer.

Resource Optimization: I centralized the loading logic for assets (Logo, Hero, Card Art) to eliminate duplicate and in-progress network requests. This reduces network costs and improves application efficiency.

Multitasking and Focus: During the ListView and Hub refactor and the Dynamic Font implementation, I maintained my regular workload, simultaneously resolving 6-8 sprint tickets in addition to the modernization work.

2 - Client / Customer / Stakeholder
I have directly improved the end-user experience by optimizing perceived performance and accessibility.

Drastic Performance Improvement (ListView): The ListView refactor transformed a poor user experience (slow scroll, flickering) into a fluid, high-performance navigation. This was achieved by eliminating manual size calculations and the inefficient caching of NSAttributedString and views (cells, headers).

Extreme Load Optimization (Wallet): The Wallet refactor (adopting UITableViewDiffableDataSource) eliminated hundreds of unnecessary network requests (100-200) that were firing for non-visible cells. The result is a dramatically faster load and response time for the customer.

Instant UI Response: The implementation of DiffableDataSource across all modules (ListView, Hub, Carousel, Shopping Hub, Thematic, Wallet, etc.) allows for instant updates (like activating offers) without reloading the entire view, improving perceived fluidity.

User Experience Enhancement: I implemented UI Prefetching in all key components (Entry, Hub, Wallet, etc.) to proactively load data. We now have more control over tasks that are in-progress or should be canceled, avoiding hundreds of unnecessary requests.

Accessibility (Dynamic Font): I led the Dynamic Font implementation, reviewing and refactoring multiple views to ensure reusability and correct constraints, making the application more accessible and inclusive.

Stakeholder Enablement (QA/Devs): I fixed a critical bug in the Wallet that caused constant crashes in the Xcode Debug View Hierarchy. This fix unblocked the Development and QA teams, allowing them to validate and debug the UI efficiently.

3 - Teamwork and Leadership
I have actively responded to feedback from 2023 and 2024 to take more initiative in mentoring, architecture, and team leadership, aligning with my 2025 goals of "Mentoring junior team members" and "Taking on next-level responsibilities".

Mentorship and Force Multiplier: I act as a technical pillar and mentor (2023 & 2024 feedback), proactively guiding my colleagues through complex architectures and providing clear solutions that unblock them, fulfilling my 2025 goal.

Initiative and Ownership: I have taken the initiative (2023 feedback) to identify and create numerous tech debt tickets, actively "promoting code improvements". I quickly resolve priority "blockers" (like crashes) to ensure sprint continuity.

Architectural Leadership and System-Wide Vision: Responding to feedback (2023 & 2024) to be "more involved in... architecture... design decisions," my understanding now extends end-to-end, enabling me to identify and resolve critical issues outside of my immediate component's scope. This was demonstrated in the global DataManager refactors and asset centralization, creating reusable components that simplify my colleagues' work and align our code with internal best practices.

Taking on Next-Level Responsibilities: My leadership in these modernization initiatives (Compositional Layouts, DiffableDataSource, Concurrency, SwiftUI) demonstrates my capacity to take on a "next-level... scope" (2023 feedback).

Future Focus: I am prepared for the "transitioning to SwiftUI" (2024 feedback) by modernizing our UIKit architectures. Additionally, I am progressing on my goal to obtain the AWS Developer Associate certification to strengthen our full-stack collaboration capabilities.




////-----------------------



Proposed Performance Goals (for 2026)
1. Lead Team Technical Upskilling and Productivity


Goal Description: To formalize my mentorship role (a 2025 goal ) and become a force multiplier for the team. I will lead architecture review sessions, proactively document best practices based on my refactors (DiffableDataSource, CompositionalLayouts), and act as the primary point of contact for unblocking junior members on complex technical challenges.

Why it's a High-Impact Goal:

It directly addresses manager feedback from 2023 ("mentoring newer team members," "documentation," "training materials" ) and 2024 ("mentoring junior team members" ).


This shifts the perception from "Juan solves problems fast" to "Juan makes the entire team solve problems faster." It demonstrates leadership and "next-level responsibilities".

2. Take Ownership of the Architectural Modernization Roadmap


Goal Description: To move beyond "Promote code improvements"  and take proactive ownership of our architecture. I will define and document the modernization roadmap for our components, including the migration strategy from UIKit to SwiftUI (a 2025 goal ). This involves designing new components before development begins and ensuring every new feature aligns with this modern architecture.

Why it's a High-Impact Goal:

It directly addresses manager feedback from 2023 ("being more involved in making and documenting more architecture and (system) design decisions" ) and 2024 ("new architecture designs... transitioning to SwiftUI" ).


This positions me not as someone who just fixes low-quality code (like the Carousel Hub), but as the architect who prevents it from being written in the first place.

3. Lead the Adoption of Strategic Technologies (Swift Concurrency & SwiftUI)

Goal Description: To become the Subject Matter Expert (SME) and lead the implementation of key Swift technologies. This includes (1) Leading the complete refactor of our networking and DataManager layers to adopt async/await and Strict Concurrency, and (2) Delivering the first production component or large-scale prototype built 100% in SwiftUI (a 2025 goal ).

Why it's a High-Impact Goal:

It combines the 2025 goals of "SwiftUI" and "Promote code improvements"  into a single, clear leadership initiative.

It demonstrates alignment with industry and company priorities (mentioned in 2024 feedback ).

4. Expand Full-Stack Impact by Optimizing Client-Server Collaboration


Goal Description: To use the knowledge gained from the "AWS Developer Associate" certification (a 2025 goal ) to expand my impact beyond iOS. I will actively collaborate with the backend team to co-design at least two new APIs, focusing on optimizing data payloads and reducing overall system latency, based on my findings from the DataManager and Wallet network optimizations.

Why it's a High-Impact Goal:

It gives a tangible business purpose to the AWS certification goal.

It demonstrates full system-design thinking ("system design" ) instead of a siloed (iOS-only) view, proving readiness for a "next-level... scope".
