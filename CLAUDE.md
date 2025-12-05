# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is "The Secret Animals Society" - a single-page pet adoption mobile web application built entirely within a single HTML file. The app provides a complete user experience for pet adoption, including a multi-step questionnaire, educational resources about pet care, donation/foster flows, and lost & found reporting.

**Key Architecture Pattern**: Single-file application using CDN-loaded React + Tailwind CSS + Lucide icons, all transpiled in-browser with Babel standalone.

## Development Setup

### Running the Application

Since this is a static HTML file with no build process, you can run it using any local web server:

```bash
# Using Python (recommended for quick testing)
python3 -m http.server 8000
# Then open: http://localhost:8000

# Using Node.js serve
npx serve .
# Then open: http://localhost:5000

# Direct file opening also works
open index.html  # macOS
# Or just double-click index.html in your file browser
```

**No installation required** - all dependencies are loaded via CDN and work directly in the browser.

## Technical Architecture

### Technology Stack (All CDN-Loaded)
- **React 18** - Functional components with hooks (`useState`, `useEffect`)
- **ReactDOM 18** - Client-side rendering
- **Babel Standalone** - In-browser JSX transpilation
- **Tailwind CSS** - Utility-first styling via CDN script
- **Lucide React** - Icon library (UMD bundle)
- **Google Fonts** - Inter font family

### Single-File Structure

The entire application is contained in `index.html` with this organization:

1. **Head Section (lines 1-37)**
   - CDN imports for all dependencies
   - Custom fonts and base styles
   - Minimal custom CSS for scrollbar styling

2. **React Application (lines 42-976)**
   - Component definitions (functions)
   - Mock data (MOCK_PETS, ANIMAL_TYPES, PERSONALITIES)
   - Main App component with routing logic
   - State management via React hooks

3. **Mount Script (lines 972-974)**
   - ReactDOM root creation and rendering

### Key Components

**Layout Components**:
- `AppHeader` - Top navigation bar with title and optional back button
- `TabBar` - Bottom navigation (Home/Location tabs)
- `SafeIcon` - Wrapper for Lucide icons with fallback rendering

**Screen Components** (Full-screen views):
- `HomeScreen` - Mission statement and entry point
- `MainMenuScreen` - Main navigation hub
- `HowToHelpScreen` - Central hub for actions and resources
- `QuestionnaireScreen` - 4-step adoption questionnaire with validation
- `MatchesScreen` - Displays pet matches based on questionnaire
- `DonateQuizScreen` - Donation amount/frequency selector
- `FosterQuizScreen` - Foster eligibility questionnaire
- `LostFoundScreen` - Lost/found pet reporting forms
- `ReportCrueltyScreen` - Confidential cruelty reporting
- `LocationScreen` - Shelter location information

**Educational Info Screens**:
- `DogBodyLanguageScreen` - Dog behavior education (custom layout)
- `CatBodyLanguageScreen` - Cat behavior education (custom layout)
- `InfoScreen` - Generic template for educational content
- `KittenInfoScreen`, `VetServicesScreen`, `PetFirstAidScreen`, `IntroducingNewPetScreen` - Specific educational screens

### State Management

**App-Level State** (in `App` component):
```javascript
const [screen, setScreen] = useState('home');        // Current screen ID
const [formData, setFormData] = useState({...});     // Questionnaire data
const [matches, setMatches] = useState([]);          // Filtered pet results
const [isLoading, setIsLoading] = useState(false);   // Loading state
```

**Navigation**: String-based screen routing via `screen` state
- Screens are identified by string IDs: `'home'`, `'menu'`, `'questionnaire'`, etc.
- Navigation handled by `handleNavigate(targetScreen)` passed to child components

### Data Model

**Mock Pets Database** (lines 58-67):
```javascript
{
  id: number,
  name: string,
  age: string,
  type: 'dog' | 'cat' | 'bird' | 'turtle',
  personality: string[],  // e.g., ['active', 'energetic']
  photo: string           // Placeholder URL
}
```

**Animal Types** (lines 69-74):
```javascript
{ id: string, name: string, iconName: string }
```

**Personalities** (lines 76-82):
```javascript
{ id: string, name: string }
```

### Matching Algorithm

Located in `handleSubmit` function (lines 910-924):
1. Filters `MOCK_PETS` by selected `animalType` (exact match)
2. Filters by `personality` array (any matching trait)
3. Shows loading screen for 2 seconds (simulated API call)
4. Navigates to `matches` screen with results

## Design System

### Color Palette
- **Pink** (`pink-500`, `pink-600`, `pink-700`) - Primary actions, adoption features
- **Purple** (`purple-500`, `purple-600`) - Donate, report cruelty
- **Teal** (`teal-600`) - Educational content
- **Gray** (`gray-50` to `gray-900`) - Neutral backgrounds, text

### Component Patterns

**Multi-Step Forms**:
- Step indicator with progress circles
- Navigation buttons (Back/Next/Submit)
- Field-level validation via `isCurrentStepValid()`
- Local state for multi-step data collection

**Selection Buttons**:
- Large touch-friendly targets (p-4)
- Active state: `border-pink-500 bg-pink-100 text-pink-700 shadow-md`
- Inactive: `border-gray-300 bg-white text-gray-700`
- Grid layouts for multiple options

**Info Cards**:
- Border-left accent: `border-l-4 border-{color}-500`
- Top border variant: `border-t-8 border-{color}-600`
- Shadow and rounded corners: `rounded-2xl shadow-xl`

### Mobile-First Design
- Container max-width: `max-w-sm` (384px)
- Fixed height: `h-[800px]` simulates mobile device
- Device chrome: Black bezel with notch simulation
- Sticky header/footer for persistent navigation

## Common Development Tasks

### Adding a New Screen

1. **Create the screen component**:
```javascript
const NewScreen = ({ onNavigate }) => (
  <div className="flex flex-col h-full">
    <AppHeader title="Screen Title" onBack={() => onNavigate('previousScreen')} />
    <main className="flex-1 overflow-y-auto p-6">
      {/* Content */}
    </main>
    <TabBar activeScreen="menu" onNavigate={onNavigate} />
  </div>
);
```

2. **Add to renderScreen switch statement** (around line 935):
```javascript
case 'newScreenId': return <NewScreen onNavigate={handleNavigate} />;
```

3. **Add navigation button** in appropriate parent screen

### Modifying the Questionnaire

The questionnaire is a 4-step form with distinct sections:
- **Step 1**: Contact info (name, email, phone) - lines 423-434
- **Step 2**: Home environment (own/rent, dwelling type, yard) - lines 436-463
- **Step 3**: Lifestyle (time alone, activity level, animal type) - lines 466-496
- **Step 4**: Personality preferences (multi-select) - lines 499-515

To add a field:
1. Add to `formData` initial state (line 903)
2. Add to appropriate Step component
3. Update validation in `isCurrentStepValid()` (lines 332-345)

### Adding Mock Pets

Edit `MOCK_PETS` array (lines 58-67):
```javascript
{
  id: 9,
  name: 'NewPet',
  age: '2 years old',
  type: 'dog',
  personality: ['calm', 'shy'],
  photo: 'https://placehold.co/300x300/COLOR/FFF?text=Name'
}
```

### Styling Best Practices

**Tailwind Utilities**:
- Spacing: Use consistent scale (p-4, p-6, space-y-4, space-y-6)
- Typography: `text-sm`, `text-lg`, `text-xl`, `text-2xl` + `font-medium/semibold/bold`
- Transitions: Always add `transition-colors` or `transition-all` to interactive elements
- Hover states: `hover:bg-{color}-700` for darker shade on buttons

**Mobile Touch Targets**:
- Minimum: `p-3` (48px total height)
- Preferred: `p-4` or `py-3 px-4` for buttons

## Educational Context

This project was designed as a student learning tool for:
- Mobile-first responsive design
- React component architecture
- Form validation and multi-step UX
- Conditional rendering and state management
- CDN-based rapid prototyping without build tools

The in-browser Babel transpilation allows students to see React/JSX working without understanding webpack, npm, or build processes.

## Limitations and Considerations

### CDN Development Tradeoffs
- **Pros**: Zero setup, instant reload, beginner-friendly
- **Cons**: No hot module replacement, large initial bundle, no TypeScript, production not optimized

### Mock Data
- All pet data is hardcoded (lines 58-67)
- No backend persistence - submissions don't save anywhere
- Matching algorithm is client-side only (no real database queries)

### Browser Compatibility
- Requires modern browser with ES6+ support
- Babel standalone requires JavaScript enabled
- No polyfills for older browsers

### Performance Notes
- Entire React app re-renders on any navigation
- No code splitting (all screens load upfront)
- Images use placeholder service (placehold.co)
- For production use, consider migrating to a build tool (Vite recommended)

## Future Enhancement Patterns

### Migrating to Build System
If converting to a proper React app:
1. Extract components into separate `.jsx` files
2. Move mock data to `src/data/pets.js`
3. Replace CDN imports with npm packages
4. Use Vite for fast dev server and optimized builds
5. Add TypeScript for type safety

### Adding Real Backend
1. Replace `MOCK_PETS` with API calls
2. Add form submission endpoints
3. Implement user authentication for saved searches
4. Add admin panel for pet management

### Accessibility Improvements
- Add ARIA labels to icon-only buttons
- Improve keyboard navigation (focus indicators)
- Add screen reader announcements for screen changes
- Ensure color contrast meets WCAG AA standards (some pink/purple borders may need adjustment)

## Testing Recommendations

Since there's no test framework:
- Manual testing across different mobile viewports (375px, 390px, 414px widths)
- Test all form validation paths in questionnaire
- Verify all navigation paths work correctly
- Check that matching algorithm produces correct results
- Test with various personality/type combinations
- Ensure all educational screens render correctly

## Notes for Claude Code

When making changes:
- Always preserve the single-file structure (don't split unless explicitly requested)
- Maintain consistent indentation (2 spaces)
- Keep component definitions in logical groupings (layout, screens, info screens)
- Update the `renderScreen` switch statement when adding new screens
- Respect the existing color scheme (pink for primary actions, purple for reports/donations)
- Ensure all new screens follow the mobile-first design constraints
