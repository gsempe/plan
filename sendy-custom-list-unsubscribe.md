---
date: 2019-06-13
tags: email
---

[Sendy](https://sendy.co/?ref=28Y1X) by default adds a `List-Unsubscribe` header that links the unsubscribe page of your Sendy installation.

If one wants to customize this header to use a custom field two files must be updated:

- includes/create/send-now.php line 568

```php
$mail->AddCustomHeader('List-Unsubscribe: <'.APP_PATH.'/unsubscribe/'.short($email).'/'.short($subscriber_list).'/'.short($campaign_id).'>');
```

- scheduled.php line 612

```php
$mail->AddCustomHeader('List-Unsubscribe: <'.$app_path.'/unsubscribe/'.short($email).'/'.short($subscriber_list).'/'.short($campaign_id).'>');
```

Sendy comes with two default fields:

* Name
* Email

Let's pretend that you add 5 more fields:

* Gender
* Country
* Platform
* Paying
* UnsubscribeUrl

And you want to use `UnsubscribeUrl` for the `List-Unsubscribe` header. `UnsubscribeUrl` custom field index is 4. If you're unsubscribe email is `unsubscribe@domain.com`, in both files `includes/create/send-now.php#568` and `scheduled.php#612` you have to replace the line with:

```php
$custom_values_array = explode('%s%', $custom_values);
$mail->AddCustomHeader("List-Unsubscribe: <mailto:unsubscribe@domain.com?subject=".$email.">,<".$custom_values_array[4].">");
```

After that restart Apache server:

```bash
sudo service apache2 restart
```


References:

* [Adding Custom Headers (List-Unsubscribe, List-Id, etc.)](https://sendy.co/forum/discussion/1027/adding-custom-headers-list-unsubscribe-list-id-etc-/)
* [List Unsubscribe Header](https://sendy.co/forum/discussion/750/list-unsubscribe-header/p1)
