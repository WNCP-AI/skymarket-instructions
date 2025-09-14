# SkyMarket Business Logic Documentation

Complete business logic and domain rules documentation for the SkyMarket drone service marketplace platform.

## Overview

SkyMarket operates as a Detroit-focused marketplace connecting consumers with drone service providers across four specialized categories. The platform implements sophisticated business rules for geographic constraints, regulatory compliance, pricing models, and service workflows.

**Core Business Model**:
- Two-sided marketplace: Consumers and Providers
- 15% platform fee on completed transactions
- Escrow-based payment system for trust and security
- Detroit Metro area geographic restriction (25-mile radius)
- FAA Part 107 compliance for all drone operations

## Service Categories

### Category Definitions

SkyMarket operates exactly **4 service categories**, each with distinct business rules:

```sql
-- Service category enumeration (immutable)
CREATE TYPE service_category AS ENUM (
  'food_delivery',   -- Restaurant and food delivery via drone
  'courier',         -- Package and document delivery
  'aerial_imaging',  -- Photography, videography, inspection services
  'site_mapping'     -- Surveying, construction site mapping, real estate
);
```

#### 1. Food Delivery (`food_delivery`)

**Purpose**: Restaurant and prepared food delivery via drone
**Target Market**: Restaurants, food services, consumers
**Typical Use Cases**:
- Hot food delivery from restaurants
- Grocery and convenience item delivery
- Catering for events and offices
- Emergency food supplies

**Business Rules**:
```typescript
interface FoodDeliveryRules {
  maxWeight: 5.5; // pounds (FAA Part 107 limit)
  maxDistance: 12; // miles (battery/safety limit)
  maxDeliveryTime: 30; // minutes
  temperatureControlRequired: boolean;
  restrictedItems: string[]; // alcohol, controlled substances
  operatingHours: {
    start: '6:00'; // 6 AM
    end: '22:00';  // 10 PM
  };
  weatherRestrictions: {
    maxWindSpeed: 15; // mph
    minVisibility: 3;  // miles
    noRainPolicy: true;
  };
}

// Validation logic
function validateFoodDelivery(booking: BookingData): ValidationResult {
  // Weight limit enforcement
  if (booking.estimated_weight > 5.5) {
    return { valid: false, error: 'Food delivery weight exceeds 5.5 lb limit' };
  }

  // Time window enforcement
  const deliveryHour = new Date(booking.scheduled_at).getHours();
  if (deliveryHour < 6 || deliveryHour >= 22) {
    return { valid: false, error: 'Food delivery only available 6 AM - 10 PM' };
  }

  // Distance validation
  const distance = calculateDistance(booking.pickup_lat, booking.pickup_lng, booking.dropoff_lat, booking.dropoff_lng);
  if (distance > 12) {
    return { valid: false, error: 'Food delivery distance exceeds 12-mile limit' };
  }

  return { valid: true };
}
```

#### 2. Courier (`courier`)

**Purpose**: Package and document delivery services
**Target Market**: Businesses, law firms, medical offices, individuals
**Typical Use Cases**:
- Legal document delivery
- Medical sample transport
- Business package delivery
- Emergency document courier

**Business Rules**:
```typescript
interface CourierRules {
  maxWeight: 10; // pounds
  maxDistance: 25; // miles (full service area)
  maxDeliveryTime: 60; // minutes
  signatureRequired: boolean;
  trackingRequired: boolean;
  operatingHours: {
    start: '7:00'; // 7 AM
    end: '19:00';  // 7 PM
  };
  priorityLevels: ['standard', 'rush', 'emergency'];
  restrictedItems: string[]; // hazmat, weapons, liquids over 1L
}

// Priority pricing multipliers
const COURIER_PRIORITY_MULTIPLIERS = {
  standard: 1.0,   // Base rate
  rush: 1.5,       // 50% premium
  emergency: 2.0   // 100% premium
} as const;

function calculateCourierPrice(basePrice: number, priority: string, distance: number): number {
  const priorityMultiplier = COURIER_PRIORITY_MULTIPLIERS[priority] || 1.0;
  const distanceMultiplier = distance > 15 ? 1.2 : 1.0; // 20% surcharge for long distance
  return basePrice * priorityMultiplier * distanceMultiplier;
}
```

#### 3. Aerial Imaging (`aerial_imaging`)

**Purpose**: Photography, videography, and inspection services
**Target Market**: Real estate, construction, events, marketing agencies
**Typical Use Cases**:
- Real estate photography and tours
- Construction site monitoring
- Event photography and videography
- Infrastructure inspection
- Marketing content creation

**Business Rules**:
```typescript
interface AerialImagingRules {
  maxAltitude: 400; // feet AGL (FAA limit)
  maxFlightTime: 25; // minutes (battery limit)
  equipmentRequirements: {
    camera4K: boolean;
    gimbalStabilization: boolean;
    obstacleAvoidance: boolean;
  };
  deliverables: {
    rawImages: boolean;
    editedImages: boolean;
    video4K: boolean;
    aerialMap: boolean;
  };
  operatingHours: {
    start: 'sunrise';
    end: 'sunset';
  };
  weatherRequirements: {
    maxWindSpeed: 12; // mph for stable imagery
    minVisibility: 5;  // miles
    clearSkies: true;  // no precipitation
  };
}

// Pricing based on deliverables complexity
function calculateImagingPrice(
  basePrice: number,
  deliverables: string[],
  flightDuration: number,
  editingRequired: boolean
): number {
  let totalPrice = basePrice;

  // Deliverable add-ons
  const deliverablePricing = {
    raw_photos: 0,      // Included in base
    edited_photos: 50,  // $50 editing fee
    video_4k: 75,       // $75 video premium
    aerial_map: 100,    // $100 mapping service
    virtual_tour: 150   // $150 tour creation
  };

  deliverables.forEach(deliverable => {
    totalPrice += deliverablePricing[deliverable] || 0;
  });

  // Flight time premium
  if (flightDuration > 15) {
    totalPrice *= 1.3; // 30% premium for extended flights
  }

  return Math.round(totalPrice * 100) / 100; // Round to cents
}
```

#### 4. Site Mapping (`site_mapping`)

**Purpose**: Surveying, construction site mapping, and real estate analysis
**Target Market**: Construction companies, surveyors, real estate developers
**Typical Use Cases**:
- Construction progress monitoring
- Land surveying and mapping
- Topographical analysis
- Real estate development planning
- Infrastructure inspection

**Business Rules**:
```typescript
interface SiteMappingRules {
  maxSiteArea: 100; // acres per flight
  maxAltitude: 400; // feet AGL
  accuracyRequirement: 2; // cm accuracy for surveying
  equipmentRequirements: {
    rtk_gps: boolean;      // Real-time kinematic GPS
    highRes_camera: boolean; // 20MP+ camera
    lidar_optional: boolean; // LiDAR for advanced mapping
  };
  deliverables: {
    orthoMosaic: boolean;    // Orthomosaic map
    digitalElevationModel: boolean; // DEM/DTM
    pointCloud: boolean;     // 3D point cloud
    surveyReport: boolean;   // Professional survey report
  };
  processingTime: {
    standard: 5;    // 5 business days
    rush: 2;        // 2 business days
    expedited: 1;   // 1 business day
  };
}

// Complex pricing based on area, accuracy, and deliverables
function calculateMappingPrice(
  siteArea: number,      // acres
  accuracyLevel: string, // 'survey' | 'commercial' | 'basic'
  deliverables: string[],
  processingSpeed: string
): number {
  const baseRatePerAcre = {
    basic: 25,       // $25/acre basic mapping
    commercial: 50,  // $50/acre commercial grade
    survey: 100      // $100/acre survey grade
  };

  let totalPrice = siteArea * (baseRatePerAcre[accuracyLevel] || 50);

  // Deliverable add-ons
  const deliverablePricing = {
    orthomosaic: 0,        // Included in base
    dem_dtm: 200,          // $200 elevation model
    point_cloud: 300,      // $300 3D point cloud
    survey_report: 500,    // $500 professional report
    cad_drawings: 400,     // $400 CAD deliverable
    volume_calculations: 150 // $150 volume analysis
  };

  deliverables.forEach(deliverable => {
    totalPrice += deliverablePricing[deliverable] || 0;
  });

  // Processing speed multipliers
  const speedMultipliers = {
    standard: 1.0,    // Base price
    rush: 1.5,        // 50% premium
    expedited: 2.0    // 100% premium
  };

  totalPrice *= speedMultipliers[processingSpeed] || 1.0;

  // Minimum charge for professional mapping
  return Math.max(totalPrice, 500); // $500 minimum
}
```

## Geographic Constraints

### Detroit Metro Area Restriction

All SkyMarket operations are constrained to the Detroit Metropolitan area with strict geographic boundaries:

```typescript
interface DetroitServiceArea {
  center: {
    lat: 42.3314; // Downtown Detroit
    lng: -83.0458;
  };
  radius: 25; // miles
  bounds: {
    north: 42.6;   // Northern boundary
    south: 42.0;   // Southern boundary
    east: -82.8;   // Eastern boundary
    west: -83.5;   // Western boundary
  };
  restrictedZones: {
    detroit_city_airport: {
      center: { lat: 42.4092, lng: -83.0093 };
      radius: 5; // miles - no fly zone
    };
    coleman_young_airport: {
      center: { lat: 42.4017, lng: -82.9888 };
      radius: 3; // miles - restricted airspace
    };
  };
}

// Geographic validation functions
export function validateDetroitBounds(lat: number, lng: number): boolean {
  const bounds = DetroitServiceArea.bounds;
  return lat >= bounds.south &&
         lat <= bounds.north &&
         lng >= bounds.west &&
         lng <= bounds.east;
}

export function calculateDistanceFromCenter(lat: number, lng: number): number {
  return haversineDistance(
    DetroitServiceArea.center.lat,
    DetroitServiceArea.center.lng,
    lat,
    lng
  );
}

export function isWithinServiceRadius(lat: number, lng: number): boolean {
  const distance = calculateDistanceFromCenter(lat, lng);
  return distance <= DetroitServiceArea.radius;
}

export function checkAirspaceRestrictions(lat: number, lng: number): AirspaceCheck {
  const restrictions = DetroitServiceArea.restrictedZones;

  for (const [zoneName, zone] of Object.entries(restrictions)) {
    const distance = haversineDistance(zone.center.lat, zone.center.lng, lat, lng);
    if (distance <= zone.radius) {
      return {
        restricted: true,
        zone: zoneName,
        distance,
        message: `Location is within ${zone.radius}-mile restricted zone around ${zoneName}`
      };
    }
  }

  return { restricted: false };
}
```

### Address Validation Implementation

```sql
-- Geographic validation trigger
CREATE OR REPLACE FUNCTION validate_detroit_bounds()
RETURNS TRIGGER AS $$
BEGIN
  -- Validate pickup location if provided
  IF NEW.pickup_lat IS NOT NULL AND NEW.pickup_lng IS NOT NULL THEN
    IF NOT (NEW.pickup_lat BETWEEN 42.0 AND 42.6 AND NEW.pickup_lng BETWEEN -83.5 AND -82.8) THEN
      RAISE EXCEPTION 'Pickup location must be within Detroit Metro area';
    END IF;

    -- Check service radius (25 miles from downtown)
    IF ST_DWithin(
      ST_SetSRID(ST_Point(NEW.pickup_lng, NEW.pickup_lat), 4326),
      ST_SetSRID(ST_Point(-83.0458, 42.3314), 4326),
      40233.6  -- 25 miles in meters
    ) = FALSE THEN
      RAISE EXCEPTION 'Pickup location exceeds 25-mile service radius from Detroit';
    END IF;
  END IF;

  -- Validate dropoff location
  IF NEW.dropoff_lat IS NOT NULL AND NEW.dropoff_lng IS NOT NULL THEN
    IF NOT (NEW.dropoff_lat BETWEEN 42.0 AND 42.6 AND NEW.dropoff_lng BETWEEN -83.5 AND -82.8) THEN
      RAISE EXCEPTION 'Dropoff location must be within Detroit Metro area';
    END IF;

    -- Check service radius
    IF ST_DWithin(
      ST_SetSRID(ST_Point(NEW.dropoff_lng, NEW.dropoff_lat), 4326),
      ST_SetSRID(ST_Point(-83.0458, 42.3314), 4326),
      40233.6  -- 25 miles in meters
    ) = FALSE THEN
      RAISE EXCEPTION 'Dropoff location exceeds 25-mile service radius from Detroit';
    END IF;
  END IF;

  -- Check airport restrictions
  IF EXISTS (
    SELECT 1 WHERE ST_DWithin(
      ST_SetSRID(ST_Point(NEW.dropoff_lng, NEW.dropoff_lat), 4326),
      ST_SetSRID(ST_Point(-83.0093, 42.4092), 4326),  -- Detroit City Airport
      8046.7  -- 5 miles in meters
    )
  ) THEN
    RAISE EXCEPTION 'Location is within restricted airspace around Detroit City Airport';
  END IF;

  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER booking_location_validation
  BEFORE INSERT OR UPDATE ON bookings
  FOR EACH ROW
  EXECUTE FUNCTION validate_detroit_bounds();
```

## Booking Lifecycle Management

### Booking Status Flow

```sql
-- Booking status enumeration
CREATE TYPE booking_status AS ENUM (
  'pending',      -- Initial state, awaiting provider acceptance
  'accepted',     -- Provider accepted, payment authorized
  'in_progress',  -- Service being performed
  'completed',    -- Service completed successfully
  'cancelled'     -- Booking cancelled by either party
);

-- Payment status enumeration
CREATE TYPE payment_status AS ENUM (
  'pending',      -- No payment attempt yet
  'paid',         -- Payment authorized or captured
  'failed',       -- Payment failed
  'refunded'      -- Payment refunded
);
```

### State Transition Rules

```typescript
interface BookingStatusTransition {
  from: BookingStatus;
  to: BookingStatus;
  allowedBy: ('consumer' | 'provider' | 'admin' | 'system')[];
  conditions: string[];
  autoActions: string[];
}

const BOOKING_TRANSITIONS: BookingStatusTransition[] = [
  // Initial booking creation
  {
    from: null,
    to: 'pending',
    allowedBy: ['consumer'],
    conditions: ['valid_listing', 'future_schedule', 'within_service_area'],
    autoActions: ['create_payment_intent', 'notify_provider']
  },

  // Provider acceptance
  {
    from: 'pending',
    to: 'accepted',
    allowedBy: ['provider'],
    conditions: ['payment_authorized', 'provider_available', 'valid_certification'],
    autoActions: ['authorize_payment', 'notify_consumer', 'schedule_reminder']
  },

  // Service commencement
  {
    from: 'accepted',
    to: 'in_progress',
    allowedBy: ['provider', 'system'],
    conditions: ['scheduled_time_reached', 'weather_acceptable', 'airspace_clear'],
    autoActions: ['start_tracking', 'notify_consumer']
  },

  // Service completion
  {
    from: 'in_progress',
    to: 'completed',
    allowedBy: ['provider'],
    conditions: ['service_delivered', 'consumer_confirmation_optional'],
    autoActions: ['capture_payment', 'release_escrow', 'enable_reviews', 'notify_parties']
  },

  // Cancellation paths
  {
    from: 'pending',
    to: 'cancelled',
    allowedBy: ['consumer', 'provider'],
    conditions: ['cancellation_policy_met'],
    autoActions: ['void_payment_intent', 'apply_cancellation_fee']
  },
  {
    from: 'accepted',
    to: 'cancelled',
    allowedBy: ['consumer', 'provider'],
    conditions: ['cancellation_policy_met', 'refund_policy_applied'],
    autoActions: ['refund_payment', 'apply_cancellation_fee', 'notify_parties']
  },
  {
    from: 'in_progress',
    to: 'cancelled',
    allowedBy: ['provider', 'admin'],
    conditions: ['service_interruption', 'safety_concern', 'weather_emergency'],
    autoActions: ['partial_refund', 'incident_report', 'notify_parties']
  }
];

// State transition validation
export async function validateBookingTransition(
  bookingId: string,
  fromStatus: BookingStatus | null,
  toStatus: BookingStatus,
  requestedBy: UserRole,
  context: Record<string, any>
): Promise<TransitionResult> {
  const transition = BOOKING_TRANSITIONS.find(
    t => t.from === fromStatus && t.to === toStatus
  );

  if (!transition) {
    return { allowed: false, reason: 'Invalid status transition' };
  }

  if (!transition.allowedBy.includes(requestedBy)) {
    return { allowed: false, reason: 'Insufficient permissions for this transition' };
  }

  // Validate conditions
  for (const condition of transition.conditions) {
    const isValid = await validateTransitionCondition(bookingId, condition, context);
    if (!isValid) {
      return { allowed: false, reason: `Condition not met: ${condition}` };
    }
  }

  return { allowed: true, autoActions: transition.autoActions };
}
```

### Automated Workflow Actions

```typescript
// Automated actions triggered by status changes
export async function executeBookingAutoActions(
  bookingId: string,
  actions: string[],
  context: Record<string, any>
): Promise<void> {
  for (const action of actions) {
    try {
      switch (action) {
        case 'create_payment_intent':
          await createPaymentIntentForBooking(bookingId, context.amount);
          break;

        case 'notify_provider':
          await sendProviderNotification(bookingId, 'new_booking');
          break;

        case 'authorize_payment':
          await authorizePayment(bookingId);
          break;

        case 'capture_payment':
          await capturePayment(bookingId);
          break;

        case 'release_escrow':
          await releaseEscrowFunds(bookingId);
          break;

        case 'apply_cancellation_fee':
          await applyCancellationFee(bookingId, context.cancellationReason);
          break;

        case 'enable_reviews':
          await enableReviewsForBooking(bookingId);
          break;

        case 'incident_report':
          await createIncidentReport(bookingId, context.reason);
          break;

        default:
          console.warn(`Unknown auto action: ${action}`);
      }
    } catch (error) {
      console.error(`Failed to execute auto action ${action}:`, error);
      // Continue with other actions - don't fail entire transition
    }
  }
}
```

## Pricing Models

### Dynamic Pricing Engine

```typescript
interface PricingFactors {
  basePrice: number;
  distanceMiles: number;
  durationMinutes: number;
  serviceCategory: ServiceCategory;
  timeOfDay: number; // 0-23
  dayOfWeek: number; // 0-6
  weatherConditions: string;
  demand: 'low' | 'medium' | 'high';
  seasonality: number; // 0.8 - 1.2 multiplier
}

export function calculateDynamicPrice(factors: PricingFactors): number {
  let price = factors.basePrice;

  // Distance-based pricing
  if (factors.distanceMiles > 5) {
    price += (factors.distanceMiles - 5) * getDistanceRate(factors.serviceCategory);
  }

  // Duration-based pricing (for aerial imaging and mapping)
  if (['aerial_imaging', 'site_mapping'].includes(factors.serviceCategory)) {
    if (factors.durationMinutes > 30) {
      price += (factors.durationMinutes - 30) * 2; // $2/minute beyond 30 min
    }
  }

  // Time-of-day multipliers
  const timeMultipliers = {
    peak_morning: 1.2,    // 7-9 AM
    peak_evening: 1.2,    // 5-7 PM
    overnight: 1.5,       // 10 PM - 6 AM (limited services)
    standard: 1.0         // Regular hours
  };

  const timeSlot = getTimeSlot(factors.timeOfDay);
  price *= timeMultipliers[timeSlot];

  // Weekend premium
  if (factors.dayOfWeek === 0 || factors.dayOfWeek === 6) {
    price *= 1.1; // 10% weekend premium
  }

  // Weather impact
  if (factors.weatherConditions === 'challenging') {
    price *= 1.15; // 15% premium for challenging weather
  }

  // Demand surge pricing
  const demandMultipliers = { low: 0.9, medium: 1.0, high: 1.3 };
  price *= demandMultipliers[factors.demand];

  // Seasonal adjustments
  price *= factors.seasonality;

  // Platform fee (15%) - added to consumer price
  const platformFee = price * 0.15;
  const totalConsumerPrice = price + platformFee;

  return Math.round(totalConsumerPrice * 100) / 100; // Round to cents
}

function getDistanceRate(category: ServiceCategory): number {
  const rates = {
    food_delivery: 1.5,    // $1.50/mile
    courier: 2.0,          // $2.00/mile
    aerial_imaging: 3.0,   // $3.00/mile
    site_mapping: 4.0      // $4.00/mile
  };
  return rates[category] || 2.0;
}
```

### Platform Fee Structure

```typescript
interface PlatformFeeStructure {
  standardRate: 0.15; // 15% of service price
  minimumFee: 5.00;   // $5 minimum platform fee
  maximumFee: 500.00; // $500 maximum platform fee (for large mapping projects)

  // Category-specific adjustments
  categoryMultipliers: {
    food_delivery: 1.0;   // Standard 15%
    courier: 1.0;         // Standard 15%
    aerial_imaging: 0.9;  // 13.5% (lower due to higher values)
    site_mapping: 0.8;    // 12% (lowest due to highest values)
  };
}

export function calculatePlatformFee(
  servicePrice: number,
  category: ServiceCategory
): { platformFee: number; providerPayout: number; consumerTotal: number } {
  const baseRate = 0.15;
  const categoryMultiplier = PlatformFeeStructure.categoryMultipliers[category];
  const adjustedRate = baseRate * categoryMultiplier;

  let platformFee = servicePrice * adjustedRate;

  // Apply min/max constraints
  platformFee = Math.max(platformFee, PlatformFeeStructure.minimumFee);
  platformFee = Math.min(platformFee, PlatformFeeStructure.maximumFee);

  const providerPayout = servicePrice - platformFee;
  const consumerTotal = servicePrice; // Consumer pays service price, platform fee comes from provider payout

  return {
    platformFee: Math.round(platformFee * 100) / 100,
    providerPayout: Math.round(providerPayout * 100) / 100,
    consumerTotal: Math.round(consumerTotal * 100) / 100
  };
}
```

## Cancellation and Refund Policies

### Time-Based Cancellation Policy

```typescript
interface CancellationPolicy {
  timeWindows: {
    immediate: { hours: 0, refundPercent: 100, feePercent: 0 };      // < 1 hour after booking
    early: { hours: 24, refundPercent: 100, feePercent: 0 };        // > 24 hours before service
    standard: { hours: 2, refundPercent: 50, feePercent: 25 };      // 2-24 hours before service
    late: { hours: 0, refundPercent: 0, feePercent: 100 };          // < 2 hours before service
  };

  emergencyRefund: {
    weather: { refundPercent: 100, feePercent: 0 };
    airspace: { refundPercent: 100, feePercent: 0 };
    equipment: { refundPercent: 75, feePercent: 0 };
    providerIllness: { refundPercent: 100, feePercent: 0 };
  };
}

export function calculateCancellationTerms(
  bookingId: string,
  scheduledTime: string,
  cancellationReason: string,
  requestTime: string = new Date().toISOString()
): CancellationTerms {
  const scheduledDate = new Date(scheduledTime);
  const requestDate = new Date(requestTime);
  const bookingDate = new Date(bookingId.slice(0, 8)); // Assuming booking ID includes timestamp

  const hoursUntilService = (scheduledDate.getTime() - requestDate.getTime()) / (1000 * 60 * 60);
  const hoursAfterBooking = (requestDate.getTime() - bookingDate.getTime()) / (1000 * 60 * 60);

  // Emergency cancellations
  if (CancellationPolicy.emergencyRefund[cancellationReason]) {
    return {
      allowed: true,
      refundPercent: CancellationPolicy.emergencyRefund[cancellationReason].refundPercent,
      feePercent: CancellationPolicy.emergencyRefund[cancellationReason].feePercent,
      reason: 'Emergency cancellation'
    };
  }

  // Time-based cancellation
  if (hoursAfterBooking <= 1) {
    return {
      allowed: true,
      refundPercent: 100,
      feePercent: 0,
      reason: 'Immediate cancellation grace period'
    };
  }

  if (hoursUntilService >= 24) {
    return {
      allowed: true,
      refundPercent: 100,
      feePercent: 0,
      reason: 'Early cancellation (>24 hours)'
    };
  }

  if (hoursUntilService >= 2) {
    return {
      allowed: true,
      refundPercent: 50,
      feePercent: 25,
      reason: 'Standard cancellation (2-24 hours)'
    };
  }

  return {
    allowed: false,
    refundPercent: 0,
    feePercent: 100,
    reason: 'Late cancellation (<2 hours) - no refund'
  };
}
```

## Provider Requirements and Verification

### Part 107 Certification Management

```sql
-- Provider compliance tracking
CREATE TABLE provider_certifications (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  provider_id UUID REFERENCES providers(id) ON DELETE CASCADE,
  certificate_type TEXT NOT NULL CHECK (certificate_type IN ('part_107', 'medical_waiver', 'special_waiver')),
  certificate_number TEXT NOT NULL UNIQUE,
  issued_date DATE NOT NULL,
  expiry_date DATE NOT NULL,
  issuing_authority TEXT DEFAULT 'FAA',
  verification_status TEXT DEFAULT 'pending' CHECK (verification_status IN ('pending', 'verified', 'rejected', 'expired')),
  verification_notes TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Automatic expiry check
CREATE OR REPLACE FUNCTION check_certification_expiry()
RETURNS TRIGGER AS $$
BEGIN
  -- Mark expired certifications
  UPDATE provider_certifications
  SET verification_status = 'expired'
  WHERE expiry_date < CURRENT_DATE
    AND verification_status = 'verified';

  -- Disable providers with expired Part 107 certificates
  UPDATE providers
  SET is_active = FALSE
  WHERE id NOT IN (
    SELECT provider_id
    FROM provider_certifications
    WHERE certificate_type = 'part_107'
      AND verification_status = 'verified'
      AND expiry_date > CURRENT_DATE
  );

  RETURN NULL;
END;
$$ LANGUAGE plpgsql;

-- Run expiry check daily
SELECT cron.schedule('check-cert-expiry', '0 6 * * *', 'SELECT check_certification_expiry();');
```

### Provider Verification Workflow

```typescript
interface ProviderVerificationProcess {
  requirements: {
    part107Certificate: boolean;
    businessInsurance: boolean;
    equipmentInspection: boolean;
    backgroundCheck: boolean;
    businessLicense: boolean; // Detroit business license
  };

  verificationSteps: {
    document_upload: 'pending' | 'completed' | 'rejected';
    faa_verification: 'pending' | 'verified' | 'failed';
    insurance_check: 'pending' | 'verified' | 'insufficient';
    equipment_validation: 'pending' | 'approved' | 'failed';
    background_screen: 'pending' | 'cleared' | 'flagged';
    final_approval: 'pending' | 'approved' | 'rejected';
  };
}

export async function processProviderVerification(
  providerId: string,
  documents: VerificationDocuments
): Promise<VerificationResult> {
  const verificationTasks = [];

  // Part 107 Certificate Verification
  if (documents.part107Certificate) {
    verificationTasks.push(
      verifyFAACertificate(documents.part107Certificate.certificateNumber)
    );
  }

  // Insurance Verification
  if (documents.insurance) {
    verificationTasks.push(
      verifyInsuranceCoverage(documents.insurance, {
        minimumCoverage: 1000000, // $1M liability
        droneOperationsCovered: true,
        detroitAreaCovered: true
      })
    );
  }

  // Equipment Inspection
  if (documents.equipmentList) {
    verificationTasks.push(
      validateDroneEquipment(documents.equipmentList, {
        faaRegistered: true,
        currentInspection: true,
        appropriateForServices: documents.intendedServices
      })
    );
  }

  const results = await Promise.allSettled(verificationTasks);

  // Determine overall verification status
  const allPassed = results.every(result =>
    result.status === 'fulfilled' && result.value.verified === true
  );

  if (allPassed) {
    await approveProvider(providerId);
    return { status: 'approved', message: 'Provider verification completed' };
  } else {
    const failures = results
      .filter(result => result.status === 'rejected' || result.value?.verified === false)
      .map(result => result.reason || result.value?.reason);

    return { status: 'rejected', message: 'Verification failed', details: failures };
  }
}
```

This comprehensive business logic documentation provides the foundation for implementing and maintaining SkyMarket's core business rules, ensuring consistent operation across all platform features while maintaining compliance with regulatory requirements and market standards.