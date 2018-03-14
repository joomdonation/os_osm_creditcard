# Events Booking creditcard base payment plugin skeleton

## How does it work
With a credit base payment plugin, registrants will enter credit card information (card number, card expiration date, cvv code, card holder name) directly on your site for processing payment. They won't be directed to payment gateway. If you use a credit card base payment plugin on your site:
  1. Your site must have SSL certificate installed
  2. You need to go to Events Booking -> Configuration, set **Activate HTTPS** config option to **Yes**.
  
## Payment plugin structure
Usually, a payment plugin will contains on XML file and one PHP file :

1. The xml file (**os_payment_plugin_name.xml**):
  * It provide basic payment plugin information (name, description, author, copyright....). For example https://github.com/joomdonation/os_eb_creditcard/blob/master/os_creditcard.xml#L3-L12
  * It defines the payment plugin parameters such as payment plugin mode, Merchant ID... These parameters will be setup by website administrator when he edit the payment plugin in Events Booking -> Payment Plugins section. See https://github.com/joomdonation/os_eb_creditcard/blob/master/os_creditcard.xml#L14-L27 to understand how it is defined

2. The PHP file (**os_payment_plugin_name.php**) which handles the payment process, payment verification... This is actually a php class (the name of the class has this format **os_plugin_name**) and it extends **RADPayment** class.
3. The payment plugin might need to have extra library (which is usually provided by the payment gateway for processing payment). If your payment plugin need a library like that, add it into a folder **plugin_name** in your payment plugin package and define it in the xml file using **folder** tag like this https://github.com/joomdonation/os_eb_redirect/blob/master/os_redirect.xml#L37
  
## The __construct method
```php
public function __construct($params, $config = array('type' => 1))
{
	parent::__construct($params, $config);

	/*if ($params->get('mode'))
	{
		$this->url = 'the_payment_gateway_url_in_live_mode';
	}
	else
	{
		$this->url = 'the_payment_gateway_url_in_test_mode';
	}*/
}
```

Please note the **$config = array('type' => 1)** code in the __construct method. It tells Events Booking that this is a credit card based payment method, and when registrants choose this method, Events Booking will dislay inputs to allow registants enter credit card information.

You don't have to write much code inside this method. Usually, you just need to call **parent::__construct($params, $config);**, define the payment gateway URL based on the payment mode (which is Test Mode or Live Mode) setup by admin in the payment plugin parameters.

## The processPayment method
This method contains code to pass credit card information and necessary billing data to the payment gateway for processing payment. Each payment gateway requires different set of parameters, so you will need to read the payment gateway manually to know which data needs to be passed to the payment gateway. Some notes:

1. You can use $this->params->get('parameter_name') to get data for parameter_name parameter which admin setup in payment plugin configuration.
2. You can access the registration information via $row object. For example, $row->id is id of the registration record, $row->first_name is billing first name, $row->last_name... is billing last name
3. The payment amount can be get via **$data['amount']** variable
4. The payment description (by default, it is EVENT_TITLE event registration) could be accessed via **$data['item_name']** variable
5. The currency of the payment (if need to be passed to the payment gateway) can be get from **$data['currency']** variable
6. The credit card information can be access via the following variables:
  
  * **$data['x_card_num']**: Credit card number
  * **$data['exp_month']**: Credit card expiration month
  * **$data['exp_year']**: Credit card expiration year
  * **$['card_holder_name']**: Credit card holder name
7. After passing data to the payment gateway for processing payment (use CURL or payment gateway libray):
  ** If the payment is success, you need to obtain Transaction ID of the payment, then call $this->onPaymentSuccess($row, $transactionId); to finish the process, and finally, redirect registrants to payment complete pate
  ** If the payment is failred, you need to obtain the reason of the error, store it in the session and redirect registrants to the registration failure page
  ** The code skeleton for this process is:
    ```php
    // Success is the result of the payment process
$success = true;

if ($success)
{
    $transactionId = 'the_id_of_the_transaction';
    $this->onPaymentSuccess($row, $transactionId);
    
	  // Redirect to the registration complete page
	  $app->redirect(JRoute::_('index.php?option=com_eventbooking&view=complete&Itemid=' . $Itemid, false, false));
}
else
{
    // Store the reason of the error so that it is being displayed on payment failure page
	  $errorReason = 'the_returned_error_message';
	  $session     = JFactory::getSession();
	  $session->set('omnipay_payment_error_reason', $errorReason);

	  // Redirect to payment failure page
	  $app->redirect(JRoute::_('index.php?option=com_eventbooking&view=failure&Itemid=' . $Itemid, false, false));
}
    ```
