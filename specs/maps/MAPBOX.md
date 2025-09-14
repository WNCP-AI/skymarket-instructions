# SkyMarket Simple Maps Documentation

Basic Mapbox integration for the SkyMarket hackathon project - focused on essential mapping features with Detroit-centered defaults.

## Overview

SkyMarket includes a simple, reusable Mapbox component for displaying interactive maps. The implementation is minimal and practical, perfect for hackathon development and MVP features.

**Current Features**:
- Interactive map centered on Detroit
- Click handling for location selection
- Basic marker placement
- Detroit-specific geographic defaults
- Responsive design

**Key Principle**: Simple, functional maps without complex features - perfect for demonstrating location-based services.

## Current Implementation

### MapboxMap Component

**Location**: `/components/map/MapboxMap.tsx`

```typescript
"use client"

import mapboxgl from "mapbox-gl"
import { useEffect, useRef } from "react"

type Props = {
  accessToken?: string
  center?: [number, number]
  zoom?: number
  onClick?: (lngLat: { lng: number; lat: number }) => void
  marker?: [number, number] | null
  className?: string
}

export default function MapboxMap({
  accessToken,
  center = [-83.0458, 42.3314], // Detroit coordinates (lng, lat)
  zoom = 11,                    // City-level zoom
  onClick,
  marker,
  className,
}: Props) {
  const mapContainer = useRef<HTMLDivElement | null>(null)
  const mapRef = useRef<mapboxgl.Map | null>(null)
  const markerRef = useRef<mapboxgl.Marker | null>(null)

  // Initialize map
  useEffect(() => {
    mapboxgl.accessToken = accessToken || process.env.NEXT_PUBLIC_MAPBOX_TOKEN || ""
    if (!mapContainer.current || mapRef.current) return

    const map = new mapboxgl.Map({
      container: mapContainer.current,
      style: "mapbox://styles/mapbox/streets-v12",
      center,
      zoom,
    })
    mapRef.current = map

    if (onClick) {
      map.on("click", (e) => {
        onClick({ lng: e.lngLat.lng, lat: e.lngLat.lat })
      })
    }

    return () => {
      map.remove()
      mapRef.current = null
    }
  }, [accessToken, center, zoom, onClick])

  // Handle marker updates
  useEffect(() => {
    if (!mapRef.current) return
    if (marker) {
      if (!markerRef.current) {
        markerRef.current = new mapboxgl.Marker()
      }
      markerRef.current.setLngLat(marker as [number, number]).addTo(mapRef.current)
    } else if (markerRef.current) {
      markerRef.current.remove()
      markerRef.current = null
    }
  }, [marker])

  return <div ref={mapContainer} className={className || "h-72 w-full rounded-md overflow-hidden"} />
}
```

### Environment Setup

**Required Environment Variable**:
```bash
# Get a free token from mapbox.com
NEXT_PUBLIC_MAPBOX_TOKEN=pk.eyJ1IjoieW91ci11c2VybmFtZSIsImEiOiJjbGZhYmNkZWYxMjM0NTY3ODkwIn0...

# Detroit service area (already configured in project)
NEXT_PUBLIC_SERVICE_AREA_CENTER_LAT=42.3314
NEXT_PUBLIC_SERVICE_AREA_CENTER_LNG=-83.0458
NEXT_PUBLIC_SERVICE_AREA_RADIUS_MILES=25
```

## Simple Usage Patterns

### 1. Basic Map Display

```tsx
import MapboxMap from '@/components/map/MapboxMap'

// Simple map with Detroit center
<MapboxMap className="h-64 w-full rounded-lg" />
```

### 2. Location Selection

```tsx
import { useState } from 'react'
import MapboxMap from '@/components/map/MapboxMap'

function LocationPicker() {
  const [selectedLocation, setSelectedLocation] = useState<{lng: number, lat: number} | null>(null)

  return (
    <div>
      <label>Click to select location</label>
      <MapboxMap
        className="h-64 w-full border rounded-md"
        onClick={setSelectedLocation}
        marker={selectedLocation ? [selectedLocation.lng, selectedLocation.lat] : null}
      />
      {selectedLocation && (
        <p>Selected: {selectedLocation.lat.toFixed(4)}, {selectedLocation.lng.toFixed(4)}</p>
      )}
    </div>
  )
}
```

### 3. Provider Location Display

```tsx
function ProviderLocationMap({ provider }: { provider: { lat: number, lng: number } }) {
  return (
    <MapboxMap
      center={[provider.lng, provider.lat]}
      zoom={13}
      marker={[provider.lng, provider.lat]}
      className="h-48 w-full rounded-md"
    />
  )
}
```

### 4. Service Area Visualization (Future)

```tsx
// Future enhancement: show service boundaries
function ServiceAreaMap() {
  return (
    <MapboxMap
      center={[-83.0458, 42.3314]}
      zoom={10}
      className="h-96 w-full"
      // Future: add circle overlay for service area
    />
  )
}
```

## Detroit Geographic Constants

### Service Area
```typescript
// Detroit Metro area bounds for validation
export const DETROIT_BOUNDS = {
  north: 42.6,
  south: 42.0,
  east: -82.8,
  west: -83.5
}

export const DETROIT_CENTER = {
  lat: 42.3314,
  lng: -83.0458
}

// Simple validation function
export function isWithinDetroitArea(lat: number, lng: number): boolean {
  return lat >= DETROIT_BOUNDS.south &&
         lat <= DETROIT_BOUNDS.north &&
         lng >= DETROIT_BOUNDS.west &&
         lng <= DETROIT_BOUNDS.east
}
```

## Current Project Integration

### Booking Flow (Currently Disabled)

The booking page has a commented-out map component:

**Location**: `/app/booking/[listingId]/page.tsx`
```tsx
{/* <div className="mt-2">
  <MapboxMap className="h-56" />
</div> */}
```

**To enable**:
```tsx
// Uncomment and enhance for location selection
<div className="mt-2">
  <label className="text-sm text-gray-600 mb-1 block">Select pickup location (optional)</label>
  <MapboxMap
    className="h-56 border rounded-md"
    onClick={(lngLat) => {
      // Handle location selection
      console.log('Selected location:', lngLat)
    }}
  />
</div>
```

## Dependencies

### Required Packages (Already Installed)

```json
{
  "dependencies": {
    "mapbox-gl": "^3.14.0",
    "@types/mapbox-gl": "^3.4.1"
  }
}
```

### Mapbox CSS Import

**Add to your CSS file or layout**:
```css
@import 'mapbox-gl/dist/mapbox-gl.css';
```

Or in your layout component:
```typescript
import 'mapbox-gl/dist/mapbox-gl.css'
```

## Error Handling

### Token Validation

```typescript
function MapWithErrorHandling() {
  const [error, setError] = useState<string | null>(null)

  useEffect(() => {
    if (!process.env.NEXT_PUBLIC_MAPBOX_TOKEN) {
      setError('Mapbox token not configured')
    }
  }, [])

  if (error) {
    return (
      <div className="h-72 w-full bg-gray-100 border border-gray-200 rounded-md flex items-center justify-center">
        <p className="text-gray-500">{error}</p>
      </div>
    )
  }

  return <MapboxMap />
}
```

### Loading State

```typescript
import dynamic from 'next/dynamic'

// Lazy load to avoid SSR issues
const MapboxMap = dynamic(() => import('@/components/map/MapboxMap'), {
  loading: () => (
    <div className="h-72 w-full bg-gray-100 rounded-md animate-pulse flex items-center justify-center">
      <p className="text-gray-500">Loading map...</p>
    </div>
  ),
  ssr: false
})
```

## Future Enhancement Ideas

*These are potential future features - focus on basic patterns above for hackathon development.*

### 1. Address Search Integration

```typescript
// FUTURE: Add geocoding for address search
// Would require additional Mapbox API integration
// For now, use click-to-select locations
```

### 2. Route Display

```typescript
// FUTURE: Show routes between pickup and dropoff
// Simple line drawing between two points
// Could use Mapbox Directions API
```

### 3. Service Area Visualization

```typescript
// FUTURE: Show Detroit service area boundary
// Simple circle or polygon overlay
// Helps users understand coverage area
```

## Getting Started

### 1. Set up Mapbox Account

1. Go to [mapbox.com](https://mapbox.com)
2. Sign up for free account
3. Get your access token
4. Add to `.env.local`:
   ```bash
   NEXT_PUBLIC_MAPBOX_TOKEN=pk.your_token_here
   ```

### 2. Add Maps to Your Pages

```tsx
import MapboxMap from '@/components/map/MapboxMap'

function MyPage() {
  return (
    <div>
      <h2>Service Location</h2>
      <MapboxMap className="h-64 w-full border rounded-lg" />
    </div>
  )
}
```

### 3. Test the Integration

```bash
npm run dev
# Navigate to your page with maps
# Should see interactive map centered on Detroit
# Click around to test click handlers
```

## Why This Simple Approach Works

**Advantages**:
- ✅ **Fast setup**: Basic maps working in minutes
- ✅ **Lightweight**: No complex features or dependencies
- ✅ **Reliable**: Simple patterns, fewer bugs
- ✅ **Flexible**: Easy to extend when needed
- ✅ **Cost-effective**: Stays within Mapbox free tier

**When to add complex features**:
- After validating core marketplace functionality
- When users need detailed routing/navigation
- When real-time tracking becomes essential
- When advanced geocoding is required

For a marketplace MVP, basic maps handle most use cases effectively:
- Show provider/service locations
- Allow location selection for bookings
- Provide geographic context for services
- Demonstrate location-based features to stakeholders

Keep it simple and functional - perfect for hackathon demonstrations and early-stage development.