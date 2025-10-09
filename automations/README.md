# Gas Rate Tracking Automations

This directory contains automations for monitoring gas rate changes and providing intelligent notifications.

## Files Overview

### `gas-rate-monitoring.yaml`
Comprehensive automation for detecting and responding to gas rate changes.

## Automations Included

### `Gas Rate Change - Notification`
**ID**: `gas_rate_change_notification`  
**Purpose**: Automatically notify when National Grid gas rates change

**Triggers:**
- When `sensor.national_grid_gas_tariff_last_updated` state changes
- When any multiscrape rate sensor becomes available after being unavailable
- When `sensor.gas_rate_change_monitor` detects significant changes

**Conditions:**
- Ensures tariff sensor has valid data (not 'unknown' or 'unavailable')
- Prevents false alerts during system startup

**Actions:**
- Sends detailed notification with:
  - Rate change timestamp
  - Current delivery, supply, and customer charge rates
  - Rate source information (automated vs manual)
  - Rate schedule details (R-3C Colonial Division)
  - Tariff version information

**Notification Details:**
```
🏭 Gas Rate Change Detected!

📊 Current Rates:
- Last Updated: [timestamp]
- Rate Schedule: R-3C - Residential Heating Colonial
- Tariff Version: [version from website]

💰 Rate Breakdown:
- Delivery Rate: $X.XXXX/CCF (source)
- Supply Rate: $X.XXXX/CCF (source)  
- Customer Charge: $XX.XX/month

🔄 Monitor Status: [automation status]
```

## Automation Features

### 🔄 **Intelligent Monitoring**
- Watches multiscrape sensors for rate updates
- Detects when rates become available after outages
- Monitors tariff version changes on National Grid website

### 📱 **Smart Notifications**
- Detailed rate breakdown in notifications
- Shows data source (automated extraction vs manual input)
- Includes rate schedule and tariff version information
- Prevents spam by filtering false positives

### 🛡️ **Robust Error Handling**
- Only triggers when valid data is available
- Handles sensor unavailability gracefully
- Provides clear status information in notifications

## Integration Points

### **Sensor Dependencies:**
- `sensor.national_grid_gas_tariff_last_updated` - Primary trigger
- `sensor.national_grid_gas_delivery_rate` - Rate data
- `sensor.national_grid_gas_supply_rate` - Rate data
- `sensor.national_grid_gas_customer_charge` - Rate data
- `sensor.gas_rate_change_monitor` - System status

### **External Dependencies:**
- Home Assistant notification service
- Multiscrape component extracting data every 2 weeks

## Customization Options

### **Notification Settings:**
Edit the automation to change:
- Notification service (current: default notification service)
- Message format and content
- Trigger conditions and timing

### **Monitoring Frequency:**
- Triggers are event-based (when rates change)
- Multiscrape runs every 2 weeks automatically
- No polling - efficient resource usage

### **Additional Automations:**
You could add automations for:
- **Monthly cost summaries** using `sensor.estimated_monthly_gas_cost`
- **High usage alerts** using `input_number.gas_usage_alert_threshold`
- **Budget tracking** comparing actual vs projected costs
- **Seasonal rate comparisons** tracking year-over-year changes

## Troubleshooting

### If notifications aren't working:
1. Check that Home Assistant notification service is configured
2. Verify multiscrape sensors are updating properly
3. Check automation traces in Home Assistant
4. Ensure `sensor.national_grid_gas_tariff_last_updated` is functioning

### If getting too many notifications:
1. Check the condition logic in the automation
2. Verify sensors aren't flapping between states
3. Consider adding delays or additional conditions

### If missing rate changes:
1. Verify multiscrape is running every 2 weeks
2. Check that National Grid website structure hasn't changed
3. Monitor multiscrape logs for extraction errors

## Rate Change Detection Logic

The automation uses a multi-layered approach:

1. **Primary**: Tariff version changes detected by multiscrape
2. **Secondary**: Individual rate sensor state changes  
3. **Fallback**: Rate monitor sensor change detection
4. **Validation**: Ensures data quality before notifications

This ensures rate changes are caught reliably while avoiding false alerts.