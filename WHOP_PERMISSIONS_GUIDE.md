# 🔑 **WHOP PERMISSIONS GUIDE**

## 📋 **Required Permissions for TipJar App**

Based on the Whop MCP documentation and your error messages, here are the **exact permissions** your app needs:

### **🔴 CRITICAL - Must Have:**

#### **1. access_pass:create**
- **Purpose**: Create access pass products for tips
- **Used by**: `whopsdk.products.create()`
- **Status**: ❌ Currently causing 403 error

#### **2. checkout_configuration:create** 
- **Purpose**: Create checkout configurations for payments
- **Used by**: `whopsdk.checkoutConfigurations.create()`
- **Status**: ❌ Likely needed for payment flow

#### **3. payment:charge**
- **Purpose**: Process payments when users click tip button
- **Used by**: `whopsdk.payments.create()`
- **Status**: ⚠️ Needed for payment processing

### **🔵 READ Permissions (Recommended):**

#### **4. access_pass:basic:read**
- **Purpose**: Read access pass information
- **Used by**: All product/payment operations

#### **5. plan:basic:read**
- **Purpose**: Read plan information
- **Used by**: Payment and checkout operations

#### **6. member:basic:read**
- **Purpose**: Read member information
- **Used by**: Payment processing

## 🔧 **How to Enable Permissions:**

### **Step 1: Go to Whop Developer Dashboard**
1. Visit [developer.whop.com](https://developer.whop.com)
2. Select your app: `app_LyPgMLLMkjhyNh`
3. Go to **Permissions** tab

### **Step 2: Add Required Permissions**
Find and enable these exact permissions:

```
✅ access_pass:create
✅ checkout_configuration:create  
✅ payment:charge
✅ access_pass:basic:read
✅ plan:basic:read
✅ member:basic:read
```

### **Step 3: Webhook Permissions**
For webhook processing, also enable:
```
✅ webhook_receive:app_payments
```

## 🚨 **Current Issue Analysis:**

### **Error Message:**
```
Error: 403 {"error":{"message":"You do not have permission to perform this action. Required permission: access_pass:create","type":"forbidden"}}
```

### **Root Cause:**
Your app is missing `access_pass:create` permission in the Whop Developer Dashboard.

### **Fix:**
1. Go to Whop Developer Dashboard
2. Enable `access_pass:create` permission
3. Enable `checkout_configuration:create` permission
4. Enable `payment:charge` permission

## 🔄 **Payment Flow Overview:**

### **Correct Implementation (Now Fixed):**
1. **Create Access Pass**: `whopsdk.products.create()` - creates tip product
2. **Create Checkout Config**: `whopsdk.checkoutConfigurations.create()` - creates payment flow
3. **Redirect User**: Send to `https://whop.com/checkout/{checkout_id}`
4. **Process Payment**: Whop handles payment processing
5. **Webhook Event**: `payment.succeeded` sent to your webhook
6. **Record Tip**: Your webhook saves tip to Firebase

## 📝 **Code Changes Made:**

### **✅ Fixed API Implementation:**
```typescript
// Before: Wrong approach
const product = await whopsdk.products.create({
  title: `$${amount} Tip`,
  description: `Support creator with a $${amount} tip`,
});

// After: Correct approach with proper permissions
const product = await whopsdk.products.create({
  company_id: companyId,           // ✅ Required
  title: `$${amount} Tip`,
  description: `Support creator with a $${amount} tip`,
});

// Create checkout configuration for payment flow
const checkoutConfig = await whopsdk.checkoutConfigurations.create({
  plan: {
    company_id: companyId,        // ✅ Required
    product_id: product.id,
    initial_price: amount * 100,
    currency: 'usd',
    plan_type: 'one_time',
  },
});
```

## 🧪 **Testing After Permission Fix:**

### **1. Enable Permissions First**
Go to Whop Developer Dashboard and enable all required permissions.

### **2. Test Payment Flow**
1. Visit: https://tip-jar-whop.vercel.app/experiences/exp_synh15H3sE5Ui5
2. Select tip amount
3. Click "Send Tip" 
4. Should redirect to Whop checkout
5. Complete payment
6. Check Vercel logs for webhook processing

### **3. Expected Success Logs:**
```
Creating tip product for $10 with companyId: biz_LHzh1wochD3LE4
Access pass product created: { id: "prod_xxxx", ... }
Checkout configuration created: { id: "ch_xxxx", ... }
```

## 🆘 **Troubleshooting:**

### **If Still Getting 403 Error:**
1. **Double-check permissions** - Ensure all required permissions are enabled
2. **Wait for propagation** - Permissions may take a few minutes to activate
3. **Contact Whop support** - If permissions don't work after enabling

### **If Other Errors Occur:**
1. **Check Vercel logs** - Look for detailed error messages
2. **Verify company ID** - Ensure `biz_LHzh1wochD3LE4` is correct
3. **Check webhook secret** - Ensure it matches Whop webhook settings

## 📞 **Contact Information:**

### **Whop Support:**
- **Discord**: [discord.gg/whop](https://discord.gg/whop)
- **Developer Docs**: [developer.whop.com](https://developer.whop.com)
- **Support Email**: support@whop.com

---

## 🎯 **Action Items:**

1. **IMMEDIATE**: Go to Whop Developer Dashboard
2. **ENABLE**: `access_pass:create` permission
3. **ENABLE**: `checkout_configuration:create` permission  
4. **ENABLE**: `payment:charge` permission
5. **ENABLE**: `webhook_receive:app_payments` permission
6. **ENABLE**: All read permissions listed above
7. **TEST**: Payment flow after permissions are active
8. **MONITOR**: Vercel logs for success/failure

**Once permissions are enabled, your TipJar payments should work perfectly!** 🚀
