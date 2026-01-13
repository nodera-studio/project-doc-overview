# Core Features Documentation - UWPF Location Matcher

## Table of Contents
1. [Data Upload Module](#data-upload-module)
2. [Matching Engine](#matching-engine)
3. [Edit & Review Module](#edit--review-module)
4. [Export & Integration Module](#export--integration-module)
5. [Session Management](#session-management)
6. [Configuration & Settings](#configuration--settings)
7. [Notification System](#notification-system)
8. [Theme Management](#theme-management)

## Data Upload Module

### Description
The entry point for all location data processing, supporting multiple input methods with real-time validation and preview capabilities.

### User Workflow
1. User selects data input method (RPP ID or file upload)
2. For RPP ID: Enter 8-digit identifier and system retrieves data
3. For file upload: Drag-and-drop or browse for Excel files
4. Preview loaded data in tabular format
5. Confirm both periods loaded before proceeding

### Technical Implementation

#### Component Structure
- **Upload Component** (`/features/upload/upload.ts`)
- **File Upload Handler** (supports .xlsx, .xls, .xlsb formats)
- **RPP ID Validator** (8-digit format validation)
- **Data Preview Grid** (AG-Grid integration)

#### Data Processing
```typescript
// RPP ID Validation
private isValidRppIdFormat(rppId: string): boolean {
  return /^\d{8}$/.test(rppId.trim());
}

// File Processing Pipeline
1. File validation (size, format)
2. Excel parsing using SheetJS
3. Data normalization
4. Validation against schema
5. Storage in session
```

#### Mock API Integration
- **Valid RPP IDs**: 
  - Previous Period: `48291537`
  - Current Period: `62847193`
- **Simulated Network Delay**: 2-3 seconds
- **Error Simulation**: 2% chance for testing

### Business Rules
- Both periods must have data before proceeding
- File size limit: 10MB
- Supported formats: XLSX, XLS, XLSB
- Required fields validation before acceptance
- Automatic data type detection and conversion

## Matching Engine

### Description
Sophisticated algorithm-based matching system that automatically identifies corresponding locations between periods with configurable confidence thresholds.

### User Workflow
1. System automatically runs matching on data load
2. View match results with confidence scores
3. Filter by match status (matched, unmatched, new, removed)
4. Review suggested matches with side-by-side comparison
5. Accept, reject, or modify matches
6. Manual linking for unmatched items

### Technical Implementation

#### Matching Algorithm
```typescript
// Multi-factor scoring system
Score Calculation:
- Name similarity: 40% weight
- Address matching: 30% weight  
- Value proximity: 20% weight
- Occupancy code: 10% weight

// Confidence Thresholds
- 90-100: Auto-accept
- 70-89: Review required
- Below 70: No match
```

#### Component Architecture
- **Match Component** (`/features/match/match.ts`)
- **Custom Cell Renderers**:
  - `MatchScoreCellRenderer`: Visual score display
  - `DifferenceCellRenderer`: Highlight changes
  - `PeriodCellRenderer`: Period indicators
  - `ChildRowRenderer`: Expandable details

#### Grid Configuration
```typescript
// AG-Grid setup for matching view
{
  masterDetail: true,
  detailRowHeight: 300,
  groupDefaultExpanded: 1,
  animateRows: true,
  pagination: true,
  paginationPageSize: 50
}
```

### Business Rules
- Automatic matching runs on 70% confidence minimum
- Manual override always available
- One-to-one matching enforced
- Unmatched items tracked separately
- Match history maintained for audit

## Edit & Review Module

### Description
Comprehensive data editing interface with validation, bulk operations, and change tracking for matched location pairs.

### User Workflow
1. Navigate through matched/unmatched locations
2. Edit individual fields with inline validation
3. Bulk operations for common changes
4. Undo/redo functionality
5. Real-time error highlighting
6. Save progress with auto-save option

### Technical Implementation

#### Edit Features
```typescript
// Change tracking
interface EditableLocation {
  originalData?: Partial<Location>;
  isModified?: boolean;
  validationErrors?: {[key: string]: string};
}

// Validation rules
- Required fields check
- Format validation (ZIP, country codes)
- Business rule validation
- Cross-field dependencies
```

#### Grid Capabilities
- **Inline Editing**: Click to edit any cell
- **Keyboard Navigation**: Excel-like shortcuts
- **Copy/Paste**: Multi-cell operations
- **Validation**: Real-time error display
- **Filtering**: Column-level filters
- **Sorting**: Multi-column sort

#### History Management
```typescript
// Undo/Redo implementation
history: {
  past: EditableLocation[][];
  present: EditableLocation[];
  future: EditableLocation[][];
}

// Maximum history: 50 states
```

### Business Rules
- Mandatory fields cannot be empty
- Value changes tracked with original values
- Validation on every change
- Auto-save every 60 seconds (configurable)
- Modified items visually highlighted

## Export & Integration Module

### Description
Flexible export system supporting multiple formats and direct integration with Munich Re's FPP system.

### User Workflow
1. Select export format (Excel, CSV, JSON)
2. Configure export options
3. Preview export data
4. For FPP transfer: Enter RPP ID and validate
5. Confirm and execute export
6. Download file or confirm system transfer

### Technical Implementation

#### Export Service
```typescript
// Export formats
exportToExcel(data: Location[]): Blob
exportToCSV(data: Location[]): string  
exportToJSON(data: Location[]): string

// FPP Integration
transferToFPP(rppId: string, data: Location[]): Observable<TransferResult>
```

#### Export Configuration
- **Column Selection**: Choose fields to export
- **Formatting Options**: Date formats, number formats
- **Grouping**: By match status or location type
- **Filters**: Export subset of data
- **Templates**: Predefined export configurations

#### FPP Transfer Process
1. RPP ID validation (8 digits)
2. Connection test to FPP system
3. Data transformation to FPP format
4. Secure transfer with encryption
5. Confirmation receipt
6. Audit log entry

### Business Rules
- Export requires completed matching process
- FPP transfer needs valid RPP ID
- Audit trail for all exports
- File naming convention enforced
- Maximum export size: 50MB

## Session Management

### Description
Comprehensive session handling with save, load, and recovery features for interrupted workflows.

### User Workflow
1. Save current work with custom name
2. View list of saved sessions
3. Load previous session
4. Continue from last checkpoint
5. Delete old sessions
6. Auto-recovery for browser crashes

### Technical Implementation

#### Save/Load Service
```typescript
// Session structure
interface SavedSession {
  id: string;
  name: string;
  timestamp: Date;
  currentStep: number;
  data: WorkflowData;
  metadata: SessionMetadata;
}

// Storage implementation
- Local Storage for persistence
- Session Storage for active work
- IndexedDB for large datasets
```

#### Auto-Save Feature
- **Interval**: Configurable (30s, 60s, 5min)
- **Trigger Events**: After significant changes
- **Recovery**: Automatic on page reload
- **Conflict Resolution**: Timestamp-based

### Business Rules
- Maximum 10 saved sessions per user
- Sessions expire after 30 days
- Auto-save enabled by default
- Recovery prompt on detecting unsaved work
- Session naming required for manual saves

## Configuration & Settings

### Description
Centralized configuration management for matching thresholds, validation rules, and system behavior.

### User Workflow
1. Access configuration modal
2. Adjust matching thresholds
3. Configure validation rules
4. Set auto-save preferences
5. Customize grid display
6. Save configuration profile

### Technical Implementation

#### Settings Service
```typescript
// Configuration structure
interface AppConfig {
  matching: {
    autoMatchThreshold: number;
    suggestThreshold: number;
    fuzzyMatchEnabled: boolean;
  };
  validation: {
    requiredFields: string[];
    customRules: ValidationRule[];
  };
  display: {
    pageSize: number;
    dateFormat: string;
    numberFormat: string;
  };
}
```

#### Configuration Options
- **Matching Thresholds**: 0-100 scale
- **Validation Rules**: Add custom validators
- **Display Preferences**: Grid layout, colors
- **Export Templates**: Save configurations
- **Keyboard Shortcuts**: Customizable

### Business Rules
- Default configuration always available
- User preferences persist across sessions
- Admin configurations override user settings
- Validation rules cannot be disabled
- Critical thresholds have minimum values

## Notification System

### Description
Real-time notification system for user feedback, errors, and system status updates.

### User Workflow
1. Receive success confirmations
2. View error details with actions
3. See progress indicators for long operations
4. Dismiss or act on notifications
5. Access notification history

### Technical Implementation

#### Notification Service
```typescript
// Notification types
enum NotificationType {
  SUCCESS = 'success',
  ERROR = 'error',
  WARNING = 'warning',
  INFO = 'info'
}

// Display methods
showSuccess(message: string, duration?: number)
showError(message: string, action?: NotificationAction)
showProgress(message: string, progress: number)
```

#### Features
- **Toast Notifications**: Non-blocking alerts
- **Snackbar**: Action-based messages
- **Progress Bars**: Long-running operations
- **Error Details**: Expandable error information
- **Queue Management**: Multiple notifications

### Business Rules
- Success messages auto-dismiss (5 seconds)
- Error messages require user dismissal
- Maximum 3 concurrent notifications
- Critical errors always displayed
- Progress updates throttled to 1/second

## Theme Management

### Description
Dynamic theme system supporting light and dark modes with customizable color schemes.

### User Workflow
1. Toggle between light/dark mode
2. System detects OS preference
3. Theme persists across sessions
4. Custom color adjustments (if enabled)
5. High contrast mode available

### Technical Implementation

#### Theme Service
```typescript
// Theme management
class ThemeService {
  isDarkModeEnabled(): boolean
  toggleDarkMode(): void
  setTheme(theme: Theme): void
  getSystemPreference(): Theme
}

// CSS Custom Properties
--primary-color
--background-color
--text-color
--border-color
--shadow-color
```

#### Implementation Details
- **CSS Variables**: Dynamic theme switching
- **Local Storage**: Theme persistence
- **Media Queries**: System preference detection
- **Transition**: Smooth theme transitions
- **Component Styling**: Scoped theme variables

### Business Rules
- Default to system preference
- User preference overrides system
- Theme changes apply immediately
- All components support theming
- Accessibility compliance maintained

## Additional Features

### Help System
- **Context-Sensitive Help**: F1 for current screen
- **User Guide**: Comprehensive documentation
- **Video Tutorials**: Step-by-step guides
- **Tooltips**: Hover information
- **Keyboard Shortcuts**: Quick reference

### Performance Monitoring
- **Load Time Tracking**: Page performance metrics
- **Operation Timing**: Process duration logging
- **Error Tracking**: Automatic error reporting
- **Usage Analytics**: Feature usage statistics
- **Performance Alerts**: Slow operation warnings

### Security Features
- **Session Timeout**: Configurable inactivity timeout
- **Data Encryption**: Local storage encryption
- **Audit Logging**: All operations tracked
- **Role-Based Access**: Feature restrictions
- **Secure Communication**: HTTPS enforcement

Each feature is designed with enterprise requirements in mind, providing robust functionality while maintaining excellent user experience and system performance. The modular architecture allows for easy enhancement and customization based on specific client needs.
