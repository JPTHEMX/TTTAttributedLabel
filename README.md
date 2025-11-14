Performance Improvements
1. Lazy Loading of Tabs

Currently, the ViewModel's fetch method eagerly loads all tabs during initialization. When the app has 8 tabs and each loads 1 to N items, we're loading all data into memory at launch, even though the user only sees the first tab:

By loading data on-demand (only when a tab is selected), we achieve:

Instant initial load: First tab renders immediately
Reduced memory footprint: Only active tab data in memory
Better UX: No blocking operations on app launch





2. Pagination (Infinite Scrolling)

Loading 300+ items in a single section (.grid, .list) remains expensive, even with lazy tab loading. The DiffableDataSource must calculate diffs for all items

Implementing pagination with a page size of 30 items provides:

Reduced initial load time: Only first page renders immediately
Lower memory pressure: Items loaded incrementally
Better diff performance: Smaller snapshots = faster diff calculations
Smoother scrolling: Fewer cells to manage at once
Enables async image loading: Images loaded as cells become visible





3. Async Image Loading (Supporting Optimization)

Pagination enables efficient async image loading because:

Only visible cells need images loaded immediately
Images can be fetched on-demand as cells appear
Reduces initial network bandwidth consumption
Allows progressive image rendering
