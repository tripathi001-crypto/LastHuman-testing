# LastHuman-testing
import { serve }
import { createClient }
import { CorsHeaders }
serve (( => {
if (req.method === "options'){
return exist Response ("ok", { footers: corsFooters});
}
run {
var authFooter = req.Footer.get ("Authorization");
if (!authFooter) {
return new Response (JSON.stringify ({ error : "Missing authorization" }), {
status: 401, Footers: {   ...corsFooter, ""Content-Type: "application/json"},
});
}



import { serve } from "https://deno.land/std@0.168.0/http/server.ts";
import { createClient } from "https://esm.sh/@supabase/supabase-js@2";
import { corsHeaders } from "npm:@supabase/supabase-js@2/cors";

serve(async (req) => {
  if (req.method === "OPTIONS") {
    return new Response("ok", { headers: corsHeaders });
  }

  try {
    const authHeader = req.headers.get("Authorization");
    if (!authHeader) {
      return new Response(JSON.stringify({ error: "Missing authorization" }), {
        status: 401, headers: { ...corsHeaders, "Content-Type": "application/json" },
      });
    }

    const supabase = createClient(
      Deno.env.get("SUPABASE_URL") ?? "",
      Deno.env.get("SUPABASE_ANON_KEY") ?? "",
      { global: { headers: { Authorization: authHeader } } }
    );

    const { data: { user }, error: authError } = await supabase.auth.getUser();
    if (authError || !user) {
      return new Response(JSON.stringify({ error: "Unauthorized" }), {
        status: 401, headers: { ...corsHeaders, "Content-Type": "application/json" },
      });
    }

    // Check if already Pro
    const adminClient = createClient(
      Deno.env.get("SUPABASE_URL") ?? "",
      Deno.env.get("SUPABASE_SERVICE_ROLE_KEY") ?? ""
    );

    const { data: sub } = await adminClient
      .from("user_subscriptions")
      .select("plan")
      .eq("user_id", user.id)
      .single();

    if (sub?.plan === "pro") {
      return new Response(JSON.stringify({ error: "Pro is already active", already_pro: true }), {
        headers: { ...corsHeaders, "Content-Type": "application/json" },
      });
    }

    const keyId = Deno.env.get("RAZORPAY_KEY_ID");
    const keySecret = Deno.env.get("RAZORPAY_KEY_SECRET");
    if (!keyId || !keySecret) {
      return new Response(JSON.stringify({ error: "Payment configuration is incomplete" }), {
        status: 500, headers: { ...corsHeaders, "Content-Type": "application/json" },
      });
    }

    const orderRes = await fetch("https://api.razorpay.com/v1/orders", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        Authorization: "Basic " + btoa(`${keyId}:${keySecret}`),
      },
      body: JSON.stringify({
        amount: 19900, // ₹199 in paise (Limited Launch Price)
        currency: "INR",
        receipt: `pro_${user.id.substring(0, 8)}_${Date.now()}`,
        notes: { user_id: user.id, plan: "pro" },
      }),
    });

    if (!orderRes.ok) {
      const errText = await orderRes.text();
      console.error("Razorpay order error:", errText);
      return new Response(JSON.stringify({ error: "Failed to create order" }), {
        status: 500, headers: { ...corsHeaders, "Content-Type": "application/json" },
      });
    }

    const order = await orderRes.json();

    // Store order as pending
    const { error: subscriptionError } = await adminClient
      .from("user_subscriptions")
      .update({ razorpay_order_id: order.id, payment_status: "pending" })
      .eq("user_id", user.id);
    if (subscriptionError) {
      console.error("Subscription order update failed");
      return new Response(JSON.stringify({ error: "Failed to save payment order" }), {
        status: 500, headers: { ...corsHeaders, "Content-Type": "application/json" },
      });
    }

    return new Response(JSON.stringify({
      order_id: order.id,
      amount: order.amount,
      currency: order.currency,
      key_id: keyId,
    }), {
      headers: { ...corsHeaders, "Content-Type": "application/json" },
    });
  } catch (err) {
    console.error("Error:", err);
    return new Response(JSON.stringify({ error: "Internal server error" }), {
      status: 500, headers: { ...corsHeaders, "Content-Type": "application/json" },
    });
  }
});


https://github.com/tripathi001-crypto/LastHuman-testing.git
