# Step 15: Simple Maps Integration - (OPTIONAL)

> **Phase 4: Core Features - Location Services**
>
> Implement basic mapping functionality using the existing Mapbox component - focused on practical features perfect for hackathon development.

## Objective

Add simple mapping features to SkyMarket using the existing MapboxMap component. Focus on essential functionality that demonstrates location-based services without complex features.

**What you'll build:**
- Enable the existing map component in booking flows
- Add location selection for service bookings
- Display provider locations on maps
- Simple validation for Detroit service area

## Context Setup

Load these specifications for mapping requirements:

```
@docs/specs/maps/MAPBOX.md
@components/map/MapboxMap.tsx
@CLAUDE.md
```

**Key principles:**
- Use existing simple MapboxMap component
- Focus on core location selection features
- Keep complexity appropriate for hackathon scope
- Detroit-centered with basic validation

## Current Implementation

### Existing MapboxMap Component

**Location**: `/components/map/MapboxMap.tsx`

The project already includes a working Mapbox component with these features:
- Interactive map centered on Detroit
- Click handling for location selection
- Marker placement and management
- Detroit-specific defaults
- Responsive design

### Current Usage (Disabled)

**Location**: `/app/booking/[listingId]/page.tsx`

```tsx
// Currently commented out:
{/* <div className="mt-2">
  <MapboxMap className="h-56" />
</div> */}
```

## Implementation Prompts

### 1. Set Up Mapbox Token

**Cursor Prompt**:
```
Set up Mapbox integration for the SkyMarket project.

**Requirements**:
- Get a free Mapbox token from mapbox.com
- Add to environment configuration
- Test basic map display

**Steps**:
1. Go to mapbox.com and sign up for free account
2. Create access token
3. Add to .env.local:
   NEXT_PUBLIC_MAPBOX_TOKEN=pk.your_token_here
4. Add Mapbox CSS import to layout or globals.css:
   @import 'mapbox-gl/dist/mapbox-gl.css';

**Verification**:
- Import and render MapboxMap component
- Should see interactive map centered on Detroit
- No console errors related to token

The MapboxMap component already exists at /components/map/MapboxMap.tsx
```

### 2. Enable Location Selection in Booking

**Cursor Prompt**:
```
Enable location selection in the booking flow using the existing MapboxMap component.

**Location**: /app/booking/[listingId]/page.tsx

**Requirements**:
- Uncomment and enhance the existing map integration
- Add location selection for pickup address (optional)
- Show selected coordinates to user
- Store coordinates in form data

**Enhancement**:
```tsx
// Replace the commented map section with:
<div className="mt-2">
  <label className="text-sm text-gray-600 mb-1 block">
    Select pickup location (optional) - Click on map
  </label>
  <MapboxMap
    className="h-56 border rounded-md"
    onClick={(lngLat) => {
      // Handle location selection
      setSelectedLocation(lngLat)
      // Update the pickup address input with coordinates
      const coordString = `${lngLat.lat.toFixed(4)}, ${lngLat.lng.toFixed(4)}`
      // You can set this in the pickup_address field for now
    }}
    marker={selectedLocation ? [selectedLocation.lng, selectedLocation.lat] : null}
  />
  {selectedLocation && (
    <p className="text-xs text-gray-500 mt-1">
      Selected: {selectedLocation.lat.toFixed(4)}, {selectedLocation.lng.toFixed(4)}
    </p>
  )}
</div>
```

**Implementation**:
- Add state for selectedLocation
- Handle map click events
- Show selected coordinates
- Keep it simple - no complex validation for now

Reference the existing MapboxMap component - it already has all needed features.
```

### 3. Add Provider Location Display

**Cursor Prompt**:
```
Add provider location display using maps.

**Create a simple ProviderLocationMap component**:

```tsx
// components/maps/ProviderLocationMap.tsx
import MapboxMap from '@/components/map/MapboxMap'

type Props = {
  location: { lat: number; lng: number }
  name?: string
  className?: string
}

export function ProviderLocationMap({ location, name, className }: Props) {
  return (
    <div>
      {name && <h3 className="text-sm font-medium mb-2">{name} Location</h3>}
      <MapboxMap
        center={[location.lng, location.lat]}
        zoom={13} // Closer zoom for provider location
        marker={[location.lng, location.lat]}
        className={className || "h-48 w-full rounded-md border"}
      />
    </div>
  )
}
```

**Usage**:
- Add to provider detail pages
- Show on service listings
- Use in provider dashboard

**Integration**:
- Import in provider pages: /app/provider/[id]/page.tsx
- Use sample Detroit coordinates for demo
- Keep styling simple and clean

The existing MapboxMap component handles all the map logic.
```

### 4. Add Simple Location Validation

**Cursor Prompt**:
```
Add basic Detroit area validation for location selection.

**Create validation utility**:

```tsx
// lib/maps/validation.ts
export const DETROIT_BOUNDS = {
  north: 42.6,
  south: 42.0,
  east: -82.8,
  west: -83.5
}

export function isWithinDetroitArea(lat: number, lng: number): boolean {
  return lat >= DETROIT_BOUNDS.south &&
         lat <= DETROIT_BOUNDS.north &&
         lng >= DETROIT_BOUNDS.west &&
         lng <= DETROIT_BOUNDS.east
}

export function validateDetroitLocation(location: { lat: number; lng: number }): {
  valid: boolean;
  message?: string;
} {
  if (!isWithinDetroitArea(location.lat, location.lng)) {
    return {
      valid: false,
      message: "Location must be within Detroit Metro service area"
    }
  }
  return { valid: true }
}
```

**Integration**:
- Use in booking location selection
- Show error message for invalid locations
- Add visual feedback (red border, toast notification)

**Keep it simple**:
- Basic coordinate bounds checking
- Clear error messages
- No complex geocoding needed for hackathon
```

### 5. Add Service Area Visualization (Optional)

**Cursor Prompt**:
```
Add simple service area visualization to show coverage area.

**Optional enhancement** - only if you have extra time:

```tsx
// components/maps/ServiceAreaMap.tsx
import MapboxMap from '@/components/map/MapboxMap'

export function ServiceAreaMap() {
  return (
    <div>
      <h3 className="text-lg font-semibold mb-2">Detroit Metro Service Area</h3>
      <p className="text-sm text-gray-600 mb-4">
        SkyMarket provides drone services within 25 miles of downtown Detroit.
      </p>
      <MapboxMap
        center={[-83.0458, 42.3314]} // Detroit center
        zoom={10} // Show wider area
        className="h-64 w-full rounded-lg border"
      />
      <p className="text-xs text-gray-500 mt-2">
        Service area: 25-mile radius from downtown Detroit
      </p>
    </div>
  )
}
```

**Usage**:
- Add to about/how-it-works page
- Show in provider onboarding
- Use on landing page to show coverage

**Future enhancement**:
- Could add circle overlay to show exact boundary
- For now, just show the map centered on Detroit
```

## Dependencies & Setup

### Required Packages (Already Installed)

```json
{
  "dependencies": {
    "mapbox-gl": "^3.14.0",
    "@types/mapbox-gl": "^3.4.1"
  }
}
```

### Environment Configuration

```bash
# Required environment variable
NEXT_PUBLIC_MAPBOX_TOKEN=pk.your_mapbox_token_here

# Detroit service area (already configured)
NEXT_PUBLIC_SERVICE_AREA_CENTER_LAT=42.3314
NEXT_PUBLIC_SERVICE_AREA_CENTER_LNG=-83.0458
NEXT_PUBLIC_SERVICE_AREA_RADIUS_MILES=25
```

### CSS Import

Add to your globals.css or layout:
```css
@import 'mapbox-gl/dist/mapbox-gl.css';
```

## Verification Steps

### 1. Basic Map Display

```bash
npm run dev

# Test basic map functionality:
# 1. Navigate to any page with MapboxMap component
# 2. Should see interactive map centered on Detroit
# 3. Should be able to pan and zoom
# 4. No console errors
```

### 2. Location Selection

```bash
# Test location selection in booking:
# 1. Go to booking page for any listing
# 2. Click on map to select location
# 3. Should see marker appear
# 4. Should show coordinates below map
# 5. Coordinates should update pickup address field
```

### 3. Provider Location Display

```bash
# Test provider location maps:
# 1. Add ProviderLocationMap to provider pages
# 2. Use sample Detroit coordinates
# 3. Should show centered map with marker
# 4. Map should be properly sized and styled
```

### 4. Validation Testing

```bash
# Test location validation:
# 1. Click locations within Detroit area (should be valid)
# 2. Try coordinates outside Detroit (should show error)
# 3. Error messages should be clear and helpful
```

## Expected Outcome

After completing this step, you'll have:

✅ **Working Maps**: Interactive maps using Mapbox GL JS
✅ **Location Selection**: Click-to-select locations in booking flow
✅ **Provider Maps**: Display provider locations on maps
✅ **Basic Validation**: Simple Detroit area boundary checking
✅ **Clean Integration**: Maps that fit well with existing UI

**Key achievements:**
- Simple, functional mapping features
- Practical location selection for bookings
- Visual demonstration of location-based services
- Appropriate scope for hackathon development
- Foundation for future mapping enhancements

## Troubleshooting

### Common Issues

**Map Not Loading**:
```bash
# Check if Mapbox token is set
echo $NEXT_PUBLIC_MAPBOX_TOKEN

# Should start with 'pk.' and be valid
# If missing, get free token from mapbox.com
```

**CSS Issues**:
```bash
# Make sure Mapbox CSS is imported
# Add to globals.css:
# @import 'mapbox-gl/dist/mapbox-gl.css';
```

**Click Events Not Working**:
```bash
# Verify onClick prop is passed to MapboxMap
# Check browser console for JavaScript errors
# Ensure map is fully loaded before handling clicks
```

**Coordinates Wrong Order**:
```bash
# Mapbox uses [lng, lat] format
# Make sure coordinates are in correct order
# Detroit: [-83.0458, 42.3314] (lng, lat)
```

### Getting Mapbox Token

1. Go to [mapbox.com](https://mapbox.com)
2. Sign up for free account (no credit card required)
3. Go to Account → Access tokens
4. Copy the default public token (starts with `pk.`)
5. Add to `.env.local` as `NEXT_PUBLIC_MAPBOX_TOKEN`

## Why This Simple Approach Works

**Advantages**:
- ✅ **Quick implementation**: Existing component ready to use
- ✅ **Proven patterns**: Simple click-to-select locations
- ✅ **Good user experience**: Visual location selection
- ✅ **Hackathon appropriate**: Right level of complexity
- ✅ **Extensible**: Easy to add features later

**Future enhancements** (after hackathon):
- Address geocoding and search
- Route visualization between locations
- Real-time provider tracking
- Service area boundary overlays
- Advanced location validation

For a marketplace prototype, basic location selection and display handles most use cases effectively while demonstrating the location-based nature of the service.

## Next Steps

In **Step 16: Simple Messaging & Status Updates**, you'll add:
- Order-based messaging between users
- Status update buttons for providers
- Simple communication without complex WebSocket setup
- Form-based interactions with immediate feedback

The location selection you've built integrates seamlessly with the booking and messaging system.

---

**Previous**: [Step 14: Booking System](./14-booking-system.md) ← | → **Next**: [Step 16: Simple Messaging & Status Updates](./16-realtime-features.md)