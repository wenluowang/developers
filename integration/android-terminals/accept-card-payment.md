# 📪 Accept Card Payment

{% stepper %}
{% step %}
### Initialize SDK

{% tabs %}
{% tab title="smartpos" %}
```
private void open(CommunicationMode mode) {
    listener = new MyPosListener();
    pos = QPOSService.getInstance(OtherActivity.this, mode);
    if (pos == null) {
        statusEditText.setText("CommunicationMode unknow");
        return;
    }
    Handler handler = new Handler(Looper.myLooper());
    pos.initListener(handler, listener);
}
```
{% endtab %}

{% tab title="mpos" %}
```
private void open(CommunicationMode mode) {
    listener = new MyPosListener();
    pos = QPOSService.getInstance(OtherActivity.this, mode);
    if (pos == null) {
        statusEditText.setText("CommunicationMode unknow");
        return;
    }
    Handler handler = new Handler(Looper.myLooper());
    pos.initListener(handler, listener);
}
```
{% endtab %}
{% endtabs %}
{% endstep %}

{% step %}
### start payment

<pre class="language-java"><code class="lang-java">pos.setAmount(amount, cashbackAmount, "156",TransactionType.GOODS);
<strong>pos.doTrade(60);
</strong></code></pre>


{% endstep %}

{% step %}
### onDoTradeResult

{% tabs %}
{% tab title="CHIP" %}
<pre class="language-java"><code class="lang-java">@Override
public void onDoTradeResult(DoTradeResult result,Hashtable&#x3C;String, String> decodeData) {
    ...
    } else if (result == DoTradeResult.ICC) {
        statusEditText.setText(getString(R.string.icc_card_inserted));
        TRACE.d("EMV ICC Start")
        <a data-footnote-ref href="#user-content-fn-1">pos.doEmvApp(EmvOption.START);</a>
    }
    ...
</code></pre>
{% endtab %}

{% tab title="NFC" %}

{% endtab %}

{% tab title="MSR" %}


```java
```
{% endtab %}
{% endtabs %}
{% endstep %}

{% step %}
### emv application selection

```java
@Override
public void onRequestSelectEmvApp(ArrayList<String> appList) {
        pos.selectEmvApp(position);   //position is the index of the chosen application
        //pos.cancelSelectEmvApp();   //Cancel the transaction
}
```


{% endstep %}

{% step %}
### pin entry

{% tabs %}
{% tab title="smartpos" %}


```java
@Override
public void onQposRequestPinResult(List<String> dataList, int offlinePinTimes) {
    //draw the pin input keyboard,after finish the keyboard,then call the below api
    pos.pinMapSync(value,30);//the value is the keyboard pin coordinate position
    
}
```
{% endtab %}

{% tab title="mpos" %}

{% endtab %}

{% tab title="CR100" %}

{% endtab %}
{% endtabs %}
{% endstep %}

{% step %}
### online authorize

```java
@Override
public void onRequestOnlineProcess(String tlv) {
   //sending online message tlv data to backend server
   response=sendTlvToServer()
   ....
   //send the received online processing result to POS
   //response should contain tag 8A (Authorisation Response Code) and tag 91 (Issuer Authentication Data) 
   pos.sendOnlineProcessResult(response);
}

```


{% endstep %}

{% step %}
### confirm payment



```java
@Override
public void onRequestTransactionResult(
    TransactionResult transactionResult) {
    if (transactionResult == TransactionResult.APPROVED) {
    } else if (transactionResult == TransactionResult.TERMINATED) {
    } else if (transactionResult == TransactionResult.DECLINED) {
    } else if (transactionResult == TransactionResult.CANCEL) {
    } else if (transactionResult == TransactionResult.CAPK_FAIL) {
    } else if (transactionResult == TransactionResult.NOT_ICC) {
    } else if (transactionResult == TransactionResult.SELECT_APP_FAIL) {
    } else if (transactionResult == TransactionResult.DEVICE_ERROR) {
    } else if (transactionResult == TransactionResult.CARD_NOT_SUPPORTED) {
    } else if (transactionResult == TransactionResult.MISSING_MANDATORY_DATA) {
    } else if (transactionResult == TransactionResult.CARD_BLOCKED_OR_NO_EMV_APPS) {
    } else if (transactionResult == TransactionResult.INVALID_ICC_DATA) {
    }        
}
}
```

\

{% endstep %}

{% step %}
### Reversal Handling

If the EMV chip card refuse the transaction, but the transaction was approved by the issuer. A reversal procedure should be initiated by the mobile app. Below callback can get the required data for doing reversal:



```java
@Override
public void onReturnReversalData(String tlv) {
    ...
}
```


{% endstep %}

{% step %}
### Error Notification



```java
@Override
        public void onError(Error errorState) {
        }
Copy to clipboardErrorCopied
```

| Enum Value             | Description                                                                                                       |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------- |
| TIMEOUT                | POS return result timed out                                                                                       |
| COM\_NOT\_AVAILABLE    | Function is not supported by firmware                                                                             |
| DEVICE\_RESET          | Reset POS status                                                                                                  |
| DEVICE\_BUSY           | Terminal is processing task，you need to wait for the callback result of this task before calling the new function |
| INPUT\_OUT\_OF\_RANGE  | The length of the input data out of the range                                                                     |
| INPUT\_INVALID\_FORMAT | The length of the input data does not match the required length                                                   |
| INPUT\_INVALID         | The length or format of the input data does not match the requirements                                            |
| AMOUNT\_OUT\_OF\_LIMIT | The length of the input data out of the range                                                                     |
| MAC\_ERROR             | Data error in the sent instruction                                                                                |
{% endstep %}
{% endstepper %}



[^1]: 
