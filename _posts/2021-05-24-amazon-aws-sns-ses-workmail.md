---
layout: post
---

https://docs.aws.amazon.com/code-samples/latest/catalog/code-catalog-ruby-example_code-sns.html

# SMS

https://docs.aws.amazon.com/sns/latest/dg/sns-mobile-phone-number-as-subscriber.html
https://railsdrop.com/2021/03/04/aws-sns-how-to-send-sms/
For sending SMS to USA numbers you need to apply for 10DLC company and use that
number as DefaultSenderID

```
# app/services/send_sms.rb
class SendSms
  OTP_MESSAGE = 'Your account verification code is: '.freeze
  def initialize(message, number)
    @message = message.to_s
    @number = number
  end

  def perform
    return Error.new('SMS Message can\'t be blank') if @message.blank?
    return Error.new('SMS Number can\'t be blank') if @number.blank?

    if Rails.env.development?
      Result.new "On development we ignore sending SMS to #{@number}"
    else
      _send_sms
    end
  end

  # https://railsdrop.com/2021/03/04/aws-sns-how-to-send-sms/
  # https://docs.aws.amazon.com/code-samples/latest/catalog/ruby-sns-sns-ruby-example-send-message.rb.html
  def _send_sms
    # Notify.message "SMS '#{@message}' send to #{@number}"
    sns_client = _set_sns_client
    # https://docs.aws.amazon.com/sns/latest/api/API_SetSMSAttributes.html
    sns_client.set_sms_attributes(
      attributes: {
        'DefaultSMSType' => 'Transactional',
        'DefaultSenderID': Const.common[:sms_default_sender_id][_country_for_number],
      }
    )
    result = sns_client.publish(phone_number: @number, message: @message)

    if result.successful?
      Result.new "Successfully sent to #{@number} message_id=#{result.try :message_id}"
    else
      Error.new "Failed to send SMS to #{@number}#{result.try :error} message_id=#{result.try :message_id}"
    end
  rescue Aws::SNS::Errors::ServiceError => e
    Error.new "ServiceError #{e.message}"
  end

  def _set_sns_client
    Aws::SNS::Client.new(
      region: Rails.application.credentials.aws_region,
      access_key_id: Rails.application.credentials.aws_access_key,
      secret_access_key: Rails.application.credentials.aws_secret_key,
    )
  end

  def _country_for_number
    phone = Phonelib.parse @number
    case phone.country
    when 'US'
      'United States'
    when 'CA'
      'Canada'
    else
      'Other'
    end
  end
end
```

```
# config/initializers/const.rb
class Const
  def self.common
    hash_or_error_if_key_does_not_exists(
      name: 'myApp',
      sms_default_sender_id: {
        'United States' => '+13.....',
        'Canada' => '+15.....',
        'Other' => '',
      },
    )
  end
```

