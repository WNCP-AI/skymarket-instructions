# Step 19: Order Management - Payment Completion and Payout System

**Phase 5: Payment Integration**

## Objective

Implement comprehensive order management system that integrates payment status with booking lifecycle, handles payment confirmation after service completion, manages disputes and refunds, and provides real-time order tracking for both consumers and providers.

**Learning Focus**: Build order tracking system with payment completion triggers, implement automatic payment confirmation system with Stripe transactions, create dispute resolution workflow, and integrate with the booking system for seamless order lifecycle management.

## Context Setup

Load the required specification documents to understand SkyMarket's order management and payment completion requirements:

```bash
# Load payment and API specifications
@docs/specs/payment/PAYMENT.md
@docs/specs/business-logic/BUSINESS-LOGIC.md
@docs/specs/api/API.md
@docs/specs/api/WEBHOOKS.md
@AGENTS.md

# Reference previous implementations
@docs/tracks/developer/steps/13-booking-system.md
@docs/tracks/developer/steps/17-stripe-setup.md
@docs/tracks/developer/steps/18-payment-flow.md
```

**Key Specifications**:
- Order lifecycle management with payment integration
- Automatic payment confirmation triggers and timing requirements
- Dispute handling and refund processing workflows
- Real-time order status tracking and notifications

## Enhanced Context7 Integration

Access comprehensive Stripe payment processing and transaction management documentation through natural language prompts. Simply add "use context7" to any implementation request to get current patterns for payments, transaction handling, payment confirmation, refund processing, dispute handling, chargeback protection, and order tracking with metadata management.

## Implementation Prompts

### 1. Order Status Management System

**Prompt**: "Using the BUSINESS-LOGIC.md specification and the booking system from steps 13-14, create an order management system that tracks the complete lifecycle from payment to service completion. Integrate payment status with booking states, implement automatic status transitions, and provide real-time updates to both consumers and providers."

**Expected Implementation**:
```typescript
// lib/orders/order-manager.ts
import { createServerClient } from '@/lib/supabase/server';
import { stripe } from '@/lib/stripe/config';

export type OrderStatus =
  | 'payment_pending'
  | 'payment_confirmed'
  | 'service_scheduled'
  | 'in_progress'
  | 'completed'
  | 'disputed'
  | 'cancelled'
  | 'refunded';

export interface OrderStatusUpdate {
  orderId: string;
  status: OrderStatus;
  notes?: string;
  metadata?: Record<string, any>;
}

export class OrderManager {
  private supabase = createServerClient();

  async updateOrderStatus({ orderId, status, notes, metadata = {} }: OrderStatusUpdate) {
    const { data: order, error: fetchError } = await this.supabase
      .from('bookings')
      .select(`
        *,
        services!inner(
          id,
          title,
          provider_id,
          providers!inner(
            id,
            business_name,
            payment_account_id,
            user_id
          )
        )
      `)
      .eq('id', orderId)
      .single();

    if (fetchError || !order) {
      throw new Error('Order not found');
    }

    // Validate status transition
    const isValidTransition = this.validateStatusTransition(order.status, status);
    if (!isValidTransition) {
      throw new Error(`Invalid status transition from ${order.status} to ${status}`);
    }

    // Update order status
    const { error: updateError } = await this.supabase
      .from('bookings')
      .update({
        status,
        updated_at: new Date().toISOString(),
        ...(notes && { provider_notes: notes }),
      })
      .eq('id', orderId);

    if (updateError) {
      throw new Error('Failed to update order status');
    }

    // Handle status-specific actions
    await this.handleStatusActions(order, status, metadata);

    // Send notifications
    await this.sendStatusNotifications(order, status);

    return { success: true };
  }

  private validateStatusTransition(currentStatus: string, newStatus: OrderStatus): boolean {
    const validTransitions: Record<string, OrderStatus[]> = {
      'pending': ['payment_confirmed', 'cancelled'],
      'payment_confirmed': ['service_scheduled', 'cancelled', 'refunded'],
      'service_scheduled': ['in_progress', 'cancelled'],
      'in_progress': ['completed', 'cancelled', 'disputed'],
      'completed': ['disputed'],
      'disputed': ['completed', 'refunded'],
      'cancelled': ['refunded'],
    };

    return validTransitions[currentStatus]?.includes(newStatus) || false;
  }

  private async handleStatusActions(order: any, status: OrderStatus, metadata: any) {
    switch (status) {
      case 'completed':
        await this.confirmPayment(order);
        break;
      case 'disputed':
        await this.handleDispute(order, metadata);
        break;
      case 'refunded':
        await this.processRefund(order, metadata);
        break;
    }
  }

  private async confirmPayment(order: any) {
    if (!order.stripe_payment_intent_id) {
      console.error('Missing payment information');
      return;
    }

    try {
      // Confirm payment completion
      const paymentIntent = await stripe.paymentIntents.retrieve(order.stripe_payment_intent_id);

      if (paymentIntent.status === 'succeeded') {
        // Record the payment completion in our system
        await this.supabase
          .from('payment_confirmations')
          .insert({
            booking_id: order.id,
            provider_id: order.services.provider_id,
            amount: paymentIntent.amount,
            currency: paymentIntent.currency,
            stripe_payment_intent_id: paymentIntent.id,
            status: 'confirmed',
            confirmed_at: new Date().toISOString(),
          });
      }
    } catch (error) {
      console.error('Failed to confirm payment:', error);
    }
  }

  private async handleDispute(order: any, metadata: any) {
    // Create dispute record
    await this.supabase
      .from('disputes')
      .insert({
        booking_id: order.id,
        reason: metadata.reason || 'Service quality issue',
        description: metadata.description || '',
        initiated_by: metadata.initiated_by || 'consumer',
        status: 'open',
        created_at: new Date().toISOString(),
      });

    // Notify admin team
    console.log(`Dispute created for order ${order.id}`);
  }

  private async processRefund(order: any, metadata: any) {
    if (!order.stripe_payment_intent_id) {
      throw new Error('No payment to refund');
    }

    try {
      const refund = await stripe.refunds.create({
        payment_intent: order.stripe_payment_intent_id,
        amount: metadata.refund_amount, // Optional partial refund
        reason: metadata.reason || 'requested_by_customer',
        metadata: {
          booking_id: order.id,
          refund_reason: metadata.description || '',
        },
      });

      // Record refund
      await this.supabase
        .from('refunds')
        .insert({
          booking_id: order.id,
          stripe_refund_id: refund.id,
          amount: refund.amount,
          reason: refund.reason,
          status: refund.status,
          created_at: new Date().toISOString(),
        });

    } catch (error) {
      console.error('Failed to process refund:', error);
      throw new Error('Refund processing failed');
    }
  }

  private async sendStatusNotifications(order: any, status: OrderStatus) {
    // Implementation for sending email/SMS notifications
    // Will be covered in Step 20: Email Integration
    console.log(`Order ${order.id} status changed to ${status}`);
  }
}
```

### 2. Provider Earnings Dashboard

**Prompt**: "Using the payment specifications and Stripe payment documentation, create a provider dashboard that shows earnings history, transaction status, and payment details for completed services. Include proper formatting for currency display and integration with the order management system."

**Expected Implementation**:
```typescript
// app/dashboard/provider/earnings/page.tsx
'use client';

import { useEffect, useState } from 'react';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Badge } from '@/components/ui/badge';
import { Button } from '@/components/ui/button';
import { formatCurrency, formatDate } from '@/lib/utils';
import {
  DollarSign,
  TrendingUp,
  Calendar,
  Clock,
  ExternalLink
} from 'lucide-react';

interface EarningsData {
  total_earnings: number;
  pending_payments: number;
  completed_payments: number;
  recent_transactions: Array<{
    id: string;
    booking_id: string;
    amount: number;
    status: string;
    confirmed_at: string | null;
    service_title: string;
    consumer_name: string;
  }>;
  upcoming_transactions: Array<{
    id: string;
    amount: number;
    expected_date: string;
    service_title: string;
  }>;
}

export default function ProviderEarningsPage() {
  const [data, setData] = useState<EarningsData | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    loadEarningsData();
  }, []);

  const loadEarningsData = async () => {
    try {
      const response = await fetch('/api/provider/earnings');
      const earningsData = await response.json();
      setData(earningsData);
    } catch (error) {
      console.error('Failed to load earnings data:', error);
    } finally {
      setLoading(false);
    }
  };

  const setupPayments = async () => {
    try {
      const response = await fetch('/api/provider/payment/setup', {
        method: 'POST',
      });
      const { url } = await response.json();
      window.location.href = url;
    } catch (error) {
      console.error('Failed to setup payments:', error);
    }
  };

  if (loading) {
    return <div>Loading earnings information...</div>;
  }

  if (!data) {
    return (
      <Card>
        <CardHeader>
          <CardTitle>Payment Setup Required</CardTitle>
        </CardHeader>
        <CardContent>
          <p className="mb-4">
            Complete your payment account setup to receive payments for your drone services.
          </p>
          <Button onClick={setupPayments}>
            Set Up Payments
          </Button>
        </CardContent>
      </Card>
    );
  }

  return (
    <div className="space-y-6">
      {/* Earnings Overview */}
      <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
        <Card>
          <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
            <CardTitle className="text-sm font-medium">Total Earnings</CardTitle>
            <DollarSign className="h-4 w-4 text-muted-foreground" />
          </CardHeader>
          <CardContent>
            <div className="text-2xl font-bold">
              {formatCurrency(data.total_earnings)}
            </div>
            <p className="text-xs text-muted-foreground">
              Lifetime earnings from completed services
            </p>
          </CardContent>
        </Card>

        <Card>
          <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
            <CardTitle className="text-sm font-medium">Pending Payments</CardTitle>
            <Clock className="h-4 w-4 text-muted-foreground" />
          </CardHeader>
          <CardContent>
            <div className="text-2xl font-bold">
              {formatCurrency(data.pending_payments)}
            </div>
            <p className="text-xs text-muted-foreground">
              Services completed, awaiting payment confirmation
            </p>
          </CardContent>
        </Card>

        <Card>
          <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
            <CardTitle className="text-sm font-medium">This Month</CardTitle>
            <TrendingUp className="h-4 w-4 text-muted-foreground" />
          </CardHeader>
          <CardContent>
            <div className="text-2xl font-bold">
              {formatCurrency(data.completed_payments)}
            </div>
            <p className="text-xs text-muted-foreground">
              Payments confirmed this month
            </p>
          </CardContent>
        </Card>
      </div>

      {/* Recent Transactions */}
      <Card>
        <CardHeader>
          <CardTitle>Recent Transactions</CardTitle>
        </CardHeader>
        <CardContent>
          <div className="space-y-4">
            {data.recent_transactions.map((transaction) => (
              <div key={transaction.id} className="flex items-center justify-between p-4 border rounded-lg">
                <div>
                  <p className="font-medium">{transaction.service_title}</p>
                  <p className="text-sm text-muted-foreground">
                    {transaction.consumer_name}
                  </p>
                  <p className="text-xs text-muted-foreground">
                    {transaction.confirmed_at ? formatDate(transaction.confirmed_at) : 'Processing'}
                  </p>
                </div>
                <div className="text-right">
                  <p className="font-bold text-lg">
                    {formatCurrency(transaction.amount)}
                  </p>
                  <Badge
                    variant={transaction.status === 'confirmed' ? 'success' : 'warning'}
                    className="text-xs"
                  >
                    {transaction.status}
                  </Badge>
                </div>
              </div>
            ))}

            {data.recent_transactions.length === 0 && (
              <p className="text-center text-muted-foreground py-8">
                No recent transactions to display
              </p>
            )}
          </div>
        </CardContent>
      </Card>

      {/* Upcoming Transactions */}
      {data.upcoming_transactions.length > 0 && (
        <Card>
          <CardHeader>
            <CardTitle className="flex items-center gap-2">
              <Calendar className="h-5 w-5" />
              Upcoming Transactions
            </CardTitle>
          </CardHeader>
          <CardContent>
            <div className="space-y-3">
              {data.upcoming_transactions.map((transaction) => (
                <div key={transaction.id} className="flex items-center justify-between p-3 bg-blue-50 rounded-lg">
                  <div>
                    <p className="font-medium">{transaction.service_title}</p>
                    <p className="text-sm text-muted-foreground">
                      Expected: {formatDate(transaction.expected_date)}
                    </p>
                  </div>
                  <p className="font-bold">
                    {formatCurrency(transaction.amount)}
                  </p>
                </div>
              ))}
            </div>
          </CardContent>
        </Card>
      )}

      {/* Payment Dashboard Link */}
      <Card>
        <CardHeader>
          <CardTitle>Manage Your Account</CardTitle>
        </CardHeader>
        <CardContent>
          <p className="text-sm text-muted-foreground mb-4">
            Access your payment dashboard to view detailed transaction information, update payment details, and manage account settings.
          </p>
          <Button variant="outline" className="flex items-center gap-2">
            <ExternalLink className="h-4 w-4" />
            Open Payment Dashboard
          </Button>
        </CardContent>
      </Card>
    </div>
  );
}
```

### 3. Dispute and Refund Management

**Prompt**: "Using the PAYMENT.md specification and Stripe dispute documentation, create a dispute handling system that allows consumers to report issues, providers to respond, and administrators to mediate. Include refund processing and proper documentation for all dispute resolution activities."

**Expected Implementation**:
```typescript
// components/orders/DisputeManager.tsx
'use client';

import { useState } from 'react';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import { Textarea } from '@/components/ui/textarea';
import { Badge } from '@/components/ui/badge';
import { AlertTriangle, MessageSquare, CheckCircle, XCircle } from 'lucide-react';

interface DisputeManagerProps {
  booking: {
    id: string;
    status: string;
    total_amount: number;
    services: {
      title: string;
      providers: {
        business_name: string;
      };
    };
  };
  userRole: 'consumer' | 'provider' | 'admin';
  existingDispute?: {
    id: string;
    reason: string;
    description: string;
    status: string;
    created_at: string;
    responses: Array<{
      id: string;
      message: string;
      created_by: string;
      created_at: string;
    }>;
  };
}

export default function DisputeManager({ booking, userRole, existingDispute }: DisputeManagerProps) {
  const [disputeReason, setDisputeReason] = useState('');
  const [disputeDescription, setDisputeDescription] = useState('');
  const [responseMessage, setResponseMessage] = useState('');
  const [loading, setLoading] = useState(false);

  const canInitiateDispute = userRole === 'consumer' &&
    booking.status === 'completed' &&
    !existingDispute;

  const initiateDispute = async () => {
    setLoading(true);
    try {
      const response = await fetch(`/api/bookings/${booking.id}/dispute`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          reason: disputeReason,
          description: disputeDescription,
        }),
      });

      if (response.ok) {
        window.location.reload(); // Refresh to show new dispute
      } else {
        throw new Error('Failed to initiate dispute');
      }
    } catch (error) {
      console.error('Dispute initiation failed:', error);
    } finally {
      setLoading(false);
    }
  };

  const respondToDispute = async () => {
    if (!existingDispute) return;

    setLoading(true);
    try {
      const response = await fetch(`/api/disputes/${existingDispute.id}/respond`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          message: responseMessage,
        }),
      });

      if (response.ok) {
        setResponseMessage('');
        window.location.reload();
      } else {
        throw new Error('Failed to respond to dispute');
      }
    } catch (error) {
      console.error('Dispute response failed:', error);
    } finally {
      setLoading(false);
    }
  };

  const resolveDispute = async (resolution: 'refund' | 'reject') => {
    if (!existingDispute || userRole !== 'admin') return;

    setLoading(true);
    try {
      const response = await fetch(`/api/disputes/${existingDispute.id}/resolve`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          resolution,
          refund_amount: resolution === 'refund' ? booking.total_amount : 0,
        }),
      });

      if (response.ok) {
        window.location.reload();
      } else {
        throw new Error('Failed to resolve dispute');
      }
    } catch (error) {
      console.error('Dispute resolution failed:', error);
    } finally {
      setLoading(false);
    }
  };

  if (existingDispute) {
    return (
      <Card>
        <CardHeader>
          <CardTitle className="flex items-center gap-2">
            <AlertTriangle className="h-5 w-5 text-yellow-600" />
            Dispute Details
            <Badge
              variant={existingDispute.status === 'resolved' ? 'success' : 'warning'}
            >
              {existingDispute.status}
            </Badge>
          </CardTitle>
        </CardHeader>
        <CardContent className="space-y-4">
          <div>
            <p className="font-medium">Reason: {existingDispute.reason}</p>
            <p className="text-sm text-muted-foreground mt-1">
              {existingDispute.description}
            </p>
            <p className="text-xs text-muted-foreground">
              Initiated: {new Date(existingDispute.created_at).toLocaleDateString()}
            </p>
          </div>

          {/* Dispute Responses */}
          <div className="space-y-3">
            <h4 className="font-medium flex items-center gap-2">
              <MessageSquare className="h-4 w-4" />
              Discussion
            </h4>
            {existingDispute.responses.map((response) => (
              <div key={response.id} className="p-3 bg-gray-50 rounded-lg">
                <p className="text-sm">{response.message}</p>
                <p className="text-xs text-muted-foreground mt-1">
                  {response.created_by} - {new Date(response.created_at).toLocaleDateString()}
                </p>
              </div>
            ))}
          </div>

          {/* Response Form */}
          {existingDispute.status === 'open' && (userRole === 'provider' || userRole === 'admin') && (
            <div className="space-y-3">
              <Textarea
                placeholder="Add your response..."
                value={responseMessage}
                onChange={(e) => setResponseMessage(e.target.value)}
              />
              <Button
                onClick={respondToDispute}
                disabled={!responseMessage.trim() || loading}
              >
                {loading ? 'Sending...' : 'Send Response'}
              </Button>
            </div>
          )}

          {/* Admin Resolution Actions */}
          {existingDispute.status === 'open' && userRole === 'admin' && (
            <div className="flex gap-2 pt-4 border-t">
              <Button
                variant="destructive"
                onClick={() => resolveDispute('refund')}
                disabled={loading}
                className="flex items-center gap-2"
              >
                <CheckCircle className="h-4 w-4" />
                Approve Refund
              </Button>
              <Button
                variant="outline"
                onClick={() => resolveDispute('reject')}
                disabled={loading}
                className="flex items-center gap-2"
              >
                <XCircle className="h-4 w-4" />
                Reject Dispute
              </Button>
            </div>
          )}
        </CardContent>
      </Card>
    );
  }

  if (canInitiateDispute) {
    return (
      <Card>
        <CardHeader>
          <CardTitle className="flex items-center gap-2">
            <AlertTriangle className="h-5 w-5 text-yellow-600" />
            Report an Issue
          </CardTitle>
        </CardHeader>
        <CardContent className="space-y-4">
          <p className="text-sm text-muted-foreground">
            If you're not satisfied with the service provided, you can initiate a dispute to request a refund.
          </p>

          <div>
            <label className="block text-sm font-medium mb-2">
              Issue Type
            </label>
            <select
              value={disputeReason}
              onChange={(e) => setDisputeReason(e.target.value)}
              className="w-full p-2 border border-gray-300 rounded-md"
            >
              <option value="">Select an issue type...</option>
              <option value="service_not_delivered">Service not delivered</option>
              <option value="poor_quality">Poor service quality</option>
              <option value="damaged_property">Property damage</option>
              <option value="safety_concerns">Safety concerns</option>
              <option value="other">Other</option>
            </select>
          </div>

          <div>
            <label className="block text-sm font-medium mb-2">
              Description
            </label>
            <Textarea
              placeholder="Please describe the issue in detail..."
              value={disputeDescription}
              onChange={(e) => setDisputeDescription(e.target.value)}
              rows={4}
            />
          </div>

          <Button
            onClick={initiateDispute}
            disabled={!disputeReason || !disputeDescription.trim() || loading}
            className="w-full"
          >
            {loading ? 'Submitting...' : 'Submit Dispute'}
          </Button>

          <p className="text-xs text-muted-foreground">
            Disputes are reviewed within 24-48 hours. You will receive updates via email.
          </p>
        </CardContent>
      </Card>
    );
  }

  return null;
}
```

### 4. Order Tracking Integration

**Prompt**: "Using the API specifications and order management system, create a comprehensive order tracking page that shows the complete order lifecycle, payment status, and provides actions for order completion, disputes, and refunds. Include real-time status updates and proper integration with the WebSocket system from previous steps."

## Code Examples

### Complete Order Tracking API

```typescript
// app/api/orders/[id]/status/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { createServerClient } from '@/lib/supabase/server';
import { OrderManager } from '@/lib/orders/order-manager';

export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const supabase = createServerClient();
  const { data: { user } } = await supabase.auth.getUser();

  if (!user) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  try {
    const { data: order, error } = await supabase
      .from('bookings')
      .select(`
        *,
        services!inner(
          id,
          title,
          description,
          base_price,
          provider_id,
          providers!inner(
            id,
            business_name,
            avatar_url,
            phone_number,
            user_id
          )
        ),
        payments(
          id,
          amount,
          status,
          platform_fee,
          provider_amount,
          created_at
        ),
        disputes(
          id,
          reason,
          description,
          status,
          created_at,
          dispute_responses(
            id,
            message,
            created_by,
            created_at
          )
        ),
        refunds(
          id,
          amount,
          reason,
          status,
          created_at
        )
      `)
      .eq('id', params.id)
      .or(`consumer_id.eq.${user.id},services.providers.user_id.eq.${user.id}`)
      .single();

    if (error || !order) {
      return NextResponse.json({ error: 'Order not found' }, { status: 404 });
    }

    // Determine user role for this order
    const userRole = order.consumer_id === user.id ? 'consumer' : 'provider';

    // Get order timeline
    const timeline = await getOrderTimeline(order);

    return NextResponse.json({
      order: {
        ...order,
        user_role: userRole,
      },
      timeline,
    });
  } catch (error) {
    console.error('Failed to fetch order status:', error);
    return NextResponse.json(
      { error: 'Failed to fetch order' },
      { status: 500 }
    );
  }
}

export async function POST(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const supabase = createServerClient();
  const { data: { user } } = await supabase.auth.getUser();

  if (!user) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  const { status, notes } = await request.json();
  const orderManager = new OrderManager();

  try {
    await orderManager.updateOrderStatus({
      orderId: params.id,
      status,
      notes,
    });

    return NextResponse.json({ success: true });
  } catch (error) {
    console.error('Failed to update order status:', error);
    return NextResponse.json(
      { error: 'Failed to update order status' },
      { status: 500 }
    );
  }
}

async function getOrderTimeline(order: any) {
  // Create timeline events based on order data
  const timeline = [];

  timeline.push({
    status: 'created',
    timestamp: order.created_at,
    title: 'Order Created',
    description: 'Booking request submitted',
  });

  if (order.payments?.length > 0) {
    timeline.push({
      status: 'paid',
      timestamp: order.payments[0].created_at,
      title: 'Payment Confirmed',
      description: `Payment of ${formatCurrency(order.payments[0].amount)} processed`,
    });
  }

  if (order.status === 'completed') {
    timeline.push({
      status: 'completed',
      timestamp: order.updated_at,
      title: 'Service Completed',
      description: 'Drone service successfully delivered',
    });
  }

  if (order.disputes?.length > 0) {
    timeline.push({
      status: 'disputed',
      timestamp: order.disputes[0].created_at,
      title: 'Dispute Opened',
      description: `Issue reported: ${order.disputes[0].reason}`,
    });
  }

  return timeline.sort((a, b) =>
    new Date(a.timestamp).getTime() - new Date(b.timestamp).getTime()
  );
}
```

### Real-time Order Status Updates

```typescript
// hooks/useOrderStatus.ts
'use client';

import { useEffect, useState } from 'react';
import { createBrowserClient } from '@/lib/supabase/client';

export function useOrderStatus(orderId: string) {
  const [orderStatus, setOrderStatus] = useState<any>(null);
  const [loading, setLoading] = useState(true);
  const supabase = createBrowserClient();

  useEffect(() => {
    // Initial load
    loadOrderStatus();

    // Subscribe to real-time updates
    const subscription = supabase
      .channel(`order-${orderId}`)
      .on(
        'postgres_changes',
        {
          event: 'UPDATE',
          schema: 'public',
          table: 'bookings',
          filter: `id=eq.${orderId}`,
        },
        (payload) => {
          setOrderStatus((prev: any) => ({
            ...prev,
            ...payload.new,
          }));
        }
      )
      .subscribe();

    return () => {
      subscription.unsubscribe();
    };
  }, [orderId]);

  const loadOrderStatus = async () => {
    try {
      const response = await fetch(`/api/orders/${orderId}/status`);
      const data = await response.json();
      setOrderStatus(data.order);
    } catch (error) {
      console.error('Failed to load order status:', error);
    } finally {
      setLoading(false);
    }
  };

  const updateStatus = async (status: string, notes?: string) => {
    try {
      const response = await fetch(`/api/orders/${orderId}/status`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ status, notes }),
      });

      if (!response.ok) {
        throw new Error('Failed to update status');
      }

      // Status will be updated via real-time subscription
      return { success: true };
    } catch (error) {
      console.error('Failed to update order status:', error);
      return { success: false, error: error.message };
    }
  };

  return {
    orderStatus,
    loading,
    updateStatus,
    refresh: loadOrderStatus,
  };
}
```

## Verification Steps

### 1. Test Order Lifecycle Management
```bash
# Test complete order flow
# 1. Create booking with payment
# 2. Provider marks service as completed
# 3. Verify automatic payment confirmation
# 4. Check transaction appears in provider dashboard
```

### 2. Test Dispute System
```bash
# Test dispute creation and resolution
# 1. Consumer initiates dispute on completed order
# 2. Provider responds to dispute
# 3. Admin resolves dispute (approve/reject)
# 4. Verify refund processing if approved
```

### 3. Test Payout Automation
```bash
# Use Stripe CLI to test payment webhooks
stripe trigger payment_intent.succeeded
stripe trigger payment_intent.payment_failed

# Verify payment status updates in provider dashboard
```

## Expected Outcomes

After completing this step, you should have:

1. **Complete Order Management System**:
   - Order lifecycle tracking from payment to completion
   - Automatic status transitions and validations
   - Real-time status updates for all parties
   - Integration with payment and booking systems

2. **Provider Payout System**:
   - Automatic payment confirmation after service completion
   - Comprehensive earnings dashboard with transaction history
   - Integration with Stripe for secure payment processing
   - Payout scheduling and status tracking

3. **Dispute Resolution System**:
   - Consumer dispute initiation with detailed reporting
   - Provider response system for dispute communication
   - Admin mediation tools with refund processing
   - Complete audit trail for all dispute activities

4. **Refund Management**:
   - Partial and full refund processing
   - Integration with Stripe refund API
   - Automatic order status updates
   - Refund tracking and reporting

## Troubleshooting

### Common Issues

**Payment Confirmation Fails**:
```typescript
// Check payment status before confirmation
const paymentIntent = await stripe.paymentIntents.retrieve(paymentIntentId);
if (paymentIntent.status !== 'succeeded') {
  throw new Error('Payment not completed');
}
```

**Status Transition Validation Errors**:
```typescript
// Add comprehensive validation
const validTransitions = {
  'pending': ['accepted', 'cancelled'],
  'accepted': ['in_progress', 'cancelled'],
  'in_progress': ['completed', 'cancelled'],
  'completed': ['disputed'],
  'disputed': ['completed', 'refunded'],
};
```

**Dispute Response Permissions**:
```typescript
// Verify user can respond to dispute
const canRespond = (userRole === 'provider' && dispute.initiated_by === 'consumer') ||
                   (userRole === 'consumer' && dispute.initiated_by === 'provider') ||
                   (userRole === 'admin');
```

### Testing Tips

1. **Test All Status Transitions**: Verify each status change triggers appropriate actions
2. **Test Payment Timing**: Ensure payment confirmation occurs only after service completion
3. **Test Dispute Scenarios**: Verify all dispute types and resolutions work correctly
4. **Monitor Webhook Delivery**: Use Stripe Dashboard to verify webhook processing
5. **Test Real-time Updates**: Verify WebSocket updates work across multiple browser tabs

## Next Steps

- **Step 20**: Implement email notification system for order updates, payment confirmations, and dispute communications
- Preview: Build the email automation system that keeps all parties informed throughout the order lifecycle with professional, branded communications

The order management system is now complete and ready for integration with the email notification system that will provide comprehensive communication throughout the order lifecycle.