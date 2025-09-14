# Expert Step 04: AI SDK Tools - Marketplace Intelligence Implementation

## Objective
Implement AI SDK tool calling patterns for SkyMarket's marketplace operations. Build intelligent tools for service search, booking assistance, provider matching, and business analytics using the Vercel AI SDK.

## What You'll Master
- AI SDK tool calling architecture and implementation patterns
- Marketplace-specific AI tools for business operations
- Tool result display and user interaction patterns
- Error handling and validation for AI tool execution
- Integration with Supabase for real-time data operations

## Prerequisites
- Completed Expert Steps 01-03 (OpenAI API, AI Elements, Advanced Components)
- Understanding of Supabase database operations and RLS policies
- Experience with TypeScript interfaces and Zod validation
- Knowledge of SkyMarket's business logic and service categories

## AI Tool Architecture for Marketplaces

### Tool Categories and Use Cases
```
Service Discovery Tools    |  Booking Assistance Tools
• searchServices()        |  • checkAvailability()
• findProviders()         |  • estimatePrice()
• compareOptions()        |  • validateRequirements()
• getRecommendations()    |  • optimizeSchedule()

Operational Tools          |  Analytics Tools
• checkWeather()          |  • getServiceMetrics()
• validateRegulations()   |  • analyzePerformance()
• calculateDistance()     |  • generateInsights()
• updateStatus()          |  • trackConversions()
```

**Implementation Principles:**
1. **Business Focus:** Tools serve specific marketplace operations
2. **Data Integration:** Direct connection to Supabase for real-time data
3. **User Experience:** Tools enhance conversation flow naturally
4. **Error Handling:** Graceful failures with helpful error messages
5. **Security First:** Respect RLS policies and user permissions

## Implementation Guide

### 1. Service Discovery Tools

**Cursor Prompt:**
```
/**
 * @cursor-prompt: Implement service discovery tools for the SkyMarket chat API:
 *
 * 1. searchServices tool at app/api/chat/route.ts:
 *    - Parameters: category, location, priceRange, availability
 *    - Query Supabase services table with filters
 *    - Respect Detroit Metro area boundaries
 *    - Return structured service data with provider info
 *    - Include ratings, availability, and pricing
 *
 * 2. findProviders tool:
 *    - Parameters: serviceCategory, experience, rating, location
 *    - Query providers table with certification checks
 *    - Filter by service capabilities and availability
 *    - Return provider profiles with recent reviews
 *
 * 3. compareOptions tool:
 *    - Parameters: serviceIds array
 *    - Fetch detailed comparison data
 *    - Include pricing, features, provider info
 *    - Generate comparison insights
 *
 * Use the existing Supabase patterns and RLS policies
 */

// Example implementation pattern:
import { tool } from 'ai'
import { z } from 'zod'
import { createServerClient } from '@/lib/supabase/server'

const searchServices = tool({
  description: 'Search for drone services in Detroit Metro area',
  parameters: z.object({
    category: z.enum(['food_delivery', 'courier', 'aerial_imaging', 'site_mapping']),
    location: z.string().describe('Detroit Metro location or address'),
    priceRange: z.object({
      min: z.number().optional(),
      max: z.number().optional(),
    }).optional(),
    availability: z.enum(['immediate', 'today', 'this_week', 'flexible']).optional(),
  }),
  execute: async ({ category, location, priceRange, availability }) => {
    const supabase = createServerClient()

    let query = supabase
      .from('services')
      .select(`
        id, title, description, price, category,
        provider:profiles!provider_id (
          business_name, rating, verified
        ),
        location, availability_status
      `)
      .eq('category', category)
      .eq('status', 'active')

    if (priceRange) {
      if (priceRange.min) query = query.gte('price', priceRange.min)
      if (priceRange.max) query = query.lte('price', priceRange.max)
    }

    const { data, error } = await query.limit(10)

    if (error) {
      return { error: 'Unable to search services at this time' }
    }

    return {
      services: data || [],
      location,
      totalFound: data?.length || 0,
      category
    }
  },
})
```

### 2. Booking Assistance Tools

**Cursor Prompt:**
```
/**
 * @cursor-prompt: Implement booking assistance tools:
 *
 * 1. checkAvailability tool:
 *    - Parameters: providerId, date, timeSlot, duration
 *    - Query provider schedules and existing bookings
 *    - Check weather conditions for outdoor services
 *    - Validate against FAA regulations and no-fly zones
 *    - Return availability with alternative suggestions
 *
 * 2. estimatePrice tool:
 *    - Parameters: serviceType, location, requirements, duration
 *    - Calculate base service price
 *    - Add location-based adjustments
 *    - Include platform fees (15%)
 *    - Factor in complexity and special requirements
 *    - Return detailed pricing breakdown
 *
 * 3. validateRequirements tool:
 *    - Parameters: serviceType, requirements, location
 *    - Check permit requirements
 *    - Validate flight safety conditions
 *    - Confirm insurance coverage
 *    - Return compliance status and recommendations
 *
 * Integrate with existing booking flow patterns
 */

const estimatePrice = tool({
  description: 'Estimate pricing for drone services including all fees',
  parameters: z.object({
    serviceCategory: z.enum(['food_delivery', 'courier', 'aerial_imaging', 'site_mapping']),
    location: z.string(),
    duration: z.number().describe('Estimated service duration in minutes'),
    complexity: z.enum(['basic', 'standard', 'complex']).optional(),
    requirements: z.array(z.string()).optional(),
  }),
  execute: async ({ serviceCategory, location, duration, complexity = 'standard', requirements = [] }) => {
    // Base pricing by category (example rates)
    const basePrices = {
      food_delivery: 25,
      courier: 35,
      aerial_imaging: 150,
      site_mapping: 200,
    }

    const basePrice = basePrices[serviceCategory]
    const complexityMultiplier = {
      basic: 1.0,
      standard: 1.2,
      complex: 1.5,
    }

    const servicePrice = basePrice * complexityMultiplier[complexity] * (duration / 60)
    const platformFee = servicePrice * 0.15 // 15% platform fee
    const total = servicePrice + platformFee

    return {
      breakdown: {
        baseService: servicePrice.toFixed(2),
        platformFee: platformFee.toFixed(2),
        total: total.toFixed(2),
      },
      factors: {
        category: serviceCategory,
        complexity,
        duration,
        location,
      },
      recommendations: [
        'Prices may vary based on provider experience and demand',
        'Weather conditions may affect scheduling and pricing',
        'Consider booking during off-peak hours for better rates'
      ]
    }
  },
})
```

### 3. Weather and Regulatory Tools

**Cursor Prompt:**
```
/**
 * @cursor-prompt: Add weather and regulatory validation tools:
 *
 * 1. checkWeather tool:
 *    - Parameters: location, date, serviceType
 *    - Integrate with weather API (OpenWeatherMap)
 *    - Check wind speed, visibility, precipitation
 *    - Validate against drone operation safety limits
 *    - Return flight suitability and recommendations
 *
 * 2. validateRegulations tool:
 *    - Parameters: location, serviceType, altitude
 *    - Check airspace restrictions (airports, no-fly zones)
 *    - Validate against FAA Part 107 regulations
 *    - Consider local Detroit area restrictions
 *    - Return compliance status and required permits
 *
 * 3. calculateDistance tool:
 *    - Parameters: origin, destination, serviceType
 *    - Calculate flight distance and time
 *    - Consider Detroit Metro boundaries
 *    - Factor in no-fly zones and restricted areas
 *    - Return route information and feasibility
 *
 * Use Detroit-specific geography and regulations
 */

const checkWeather = tool({
  description: 'Check weather conditions for drone operations',
  parameters: z.object({
    location: z.string(),
    date: z.string().optional(),
    serviceType: z.enum(['food_delivery', 'courier', 'aerial_imaging', 'site_mapping']),
  }),
  execute: async ({ location, date = new Date().toISOString(), serviceType }) => {
    // In a real implementation, integrate with weather API
    // For demo purposes, return mock data with business logic

    const mockWeatherData = {
      windSpeed: Math.floor(Math.random() * 25) + 5, // 5-30 mph
      visibility: Math.floor(Math.random() * 8) + 3, // 3-10 miles
      precipitation: Math.random() > 0.7,
      temperature: Math.floor(Math.random() * 40) + 40, // 40-80°F
    }

    const isSuitableForFlight =
      mockWeatherData.windSpeed <= 20 &&
      mockWeatherData.visibility >= 3 &&
      !mockWeatherData.precipitation

    const recommendations = []
    if (mockWeatherData.windSpeed > 15) {
      recommendations.push('High winds may affect small package delivery')
    }
    if (mockWeatherData.visibility < 5) {
      recommendations.push('Reduced visibility requires extra caution')
    }
    if (mockWeatherData.precipitation) {
      recommendations.push('Postpone flight due to precipitation')
    }

    return {
      suitable: isSuitableForFlight,
      conditions: mockWeatherData,
      location,
      serviceType,
      recommendations,
      alternatives: isSuitableForFlight ? [] : [
        'Schedule for later in the day',
        'Consider weather-protected services',
        'Use ground-based alternatives if available'
      ]
    }
  },
})
```

### 4. Integration with Chat API

**Enhanced Chat Route Implementation:**
```typescript
// app/api/chat/route.ts - Enhanced with tools
import { streamText } from 'ai'
import { openai } from '@ai-sdk/openai'

const tools = {
  searchServices,
  estimatePrice,
  checkWeather,
  // ... other tools
}

export async function POST(req: Request) {
  const { messages } = await req.json()

  const result = streamText({
    model: openai('gpt-4o-mini'),
    system: systemPrompt + docsContext,
    messages: convertToCoreMessages(messages),
    tools, // Enable all marketplace tools
    maxToolRoundtrips: 3, // Allow multi-step reasoning
  })

  return result.toDataStreamResponse()
}
```

## Expert Implementation Tasks

### Tool Development
- [ ] Implement service search with real Supabase queries
- [ ] Build pricing estimation with business logic
- [ ] Add weather validation for drone operations
- [ ] Create regulatory compliance checking

### Error Handling
- [ ] Graceful fallbacks when tools fail
- [ ] Validation of tool parameters with Zod
- [ ] User-friendly error messages
- [ ] Logging and monitoring of tool usage

### UI Integration
- [ ] Tool result display components
- [ ] Loading states during tool execution
- [ ] Action buttons for follow-up operations
- [ ] Rich formatting for structured data

### Advanced Patterns
- [ ] Tool chaining for complex workflows
- [ ] Context preservation across tool calls
- [ ] User preference learning and adaptation
- [ ] Analytics tracking for tool effectiveness

## Key Implementation Insights

1. **Tool Composition**: Each tool should do one thing well and compose with others
2. **Error Resilience**: Tools should gracefully handle API failures and data issues
3. **User Experience**: Tool results should integrate seamlessly into conversation flow
4. **Performance**: Consider caching and optimizing database queries
5. **Security**: Always respect user permissions and data access policies

## Next Steps

With AI tools implemented, you can create sophisticated conversational flows that guide users through service discovery, booking, and support processes naturally.

---

**Next Step**: [Step 5: Cursor AI Development Mastery](./05-cursor-mastery.md)

**Focus**: Advanced prompting techniques and development workflows using AI assistance