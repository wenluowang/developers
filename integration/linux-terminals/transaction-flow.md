# Transaction Flow

Here's an explanation of the application and payment transaction flow with example code from the project:

#### Application Flow Overview

1. **Initialization**:
   * UI components are created using LVGL
   * Transaction buffer is initialized
   * Supported card types are configured
2. **Payment Initiation**:

```c
void Trans_Payment()
{
    if(g_pTransBuffer) {
        free(g_pTransBuffer);
        g_pTransBuffer=NULL;
    }

    g_pTransBuffer=malloc(TRANS_POOL_MAX);
    glb_trans_type = REQ_TRANS_SALE;
    PaymentInit(g_pTransBuffer);
    Enter_Amount();  // Shows amount entry UI
}
```

#### Payment Transaction Flow

1. **Amount Entry**:

```c
static void Payment_cb(lv_event_t * event) {
    if(indev_type==LV_INDEV_TYPE_KEYPAD && code==LV_EVENT_KEY) {
        switch(key) {
            case LV_KEY_ENTER:
                if(strlen((char*)txt) && atol((char*)txt) > 0) {
                    T_U8_VIEW uvAmt={(char*)txt,strlen(txt)};
                    FormatAmount(uvAmt);
                    EventRegister(EVENT_PAYMENT); // Triggers payment start
                }
                break;
        }
    }
}
```

2. **Card Detection & Processing**:

```c
void Start_Payment(void) {
    StartTrading(g_pTransBuffer); // Begins card detection
}

// Card type handling
void setTransInitData(pvoid param) {
    const pStartTransaction_t pStartTransactionData = (pStartTransaction_t)param;
    lCardsSupported = (u32)pStartTransactionData->type; // Sets supported card types
}
```

3. **Transaction Processing**:

```c
void emvOnlineProcess(unsigned char *pTransCallbackBuffer) {
    Callback_Type_t result_type = Get_callback_type(pTransCallbackBuffer);
    GetCardData(pTransCallbackBuffer);

    switch(result_type) {
        case CALLBACK_TRANS_MAG: // Magnetic stripe
            GuiEventRegister(LCD_DISP_GO_ONLINE);
            Rc = online_common(g_pTransBuffer);
            break;
            
        case CALLBACK_TRANS_ICC_ONLINE: // EMV chip
            GuiEventRegister(LCD_DISP_GO_ONLINE);
            Rc = online_common(g_pTransBuffer);
            if(Rc==RC_SUCCESS) {
                GuiEventRegister(LCD_DISP_TRANS_SUCCESS);
            } else {
                GuiEventRegister(LCD_DISP_TRANS_FAIL);
            }
            break;
    }
}
```

4. **Result Handling**:

```c
void emvTransResult(App_Rc_Code_t reCode) {
    switch(reCode) {
        case APP_RC_COMPLETED:
            GuiEventRegister(LCD_DISP_TRADE_SUCCESS);
            break;
        case APP_RC_CANCEL:
            GuiEventRegister(LCD_DISP_CANCEL);
            break;
        case APP_RC_EMV_DENAIL:
            GuiEventRegister(LCD_DISP_DECLINED_DISP);
            break;
    }
}
```

#### Key Components:

1. **Transaction Buffer** (g\_pTransBuffer):
   * Stores all transaction data in TLV format
   * Used throughout the payment flow
2. **Card Types**:

```c
enum {
    TRANS_CARD_MAG=0x01,  // Magnetic
    TRANS_CARD_ICC,       // EMV Chip
    TRANS_CARD_CTLS       // Contactless
};
```

3. **UI Integration**:
   * Uses LVGL for all UI components
   * Event-driven architecture with GuiEventRegister calls

The flow is completely event-driven, with the UI layer communicating with the transaction processing layer through events and callbacks, while maintaining transaction state in the shared buffer.
