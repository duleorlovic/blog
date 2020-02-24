---
layout: post
---

Demo https://console.aws.amazon.com/textract/home?region=us-east-1#/demo
Example app demo for redact https://youtu.be/Hth17MmxwtQ?t=521
https://docs.aws.amazon.com/textract/latest/dg/other-examples.html
Sample python apps
https://github.com/aws-samples/amazon-textract-code-samples/tree/master/python
and full
https://github.com/aws-samples/amazon-textract-textractor


# Python boto3

```
python3 -m pip install boto3

python3
import boto3
```

# Ruby aws-sdk

https://docs.aws.amazon.com/sdk-for-ruby/v3/api/
https://docs.aws.amazon.com/sdk-for-ruby/v3/developer-guide/welcome.html
https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/S3/Resource.html
```
irb
require 'aws-sdk-s3'  # v2: require 'aws-sdk'

s3 = Aws::S3::Resource.new(region: 'us-east-1')

s3.buckets.limit(50).each do |b|
  puts "#{b.name}"
end
bucket = s3.bucket bucket
bucket.map {|o| o.public_url}
```

Also SNS publish https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/SNS/Client.html#publish-instance_method
```
require 'aws-sdk-sns'
client = Aws::SNS::Client.new
resp = client.publish(
  topic_arn: topic_arn,
  message: "message", # required
)
```

For https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/Textract.html
and client
https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/Textract/Client.html#detect_document_text-instance_method
and particular type
https://docs.aws.amazon.com/textract/latest/dg/API_DetectDocumentText.html

```
require 'aws-sdk-textract'
documentName = "simple-document-image.jpg"
file = File.open(documentName)
response = client.detect_document_text({
  document: { # required
    bytes: file.read,
    # s3_object: {
    #   bucket: "ki-textract-demo-docs",
    #   name: "simple-document-image.jpg",
    #   version: "S3ObjectVersion",
    # },
  },
})

response.document_metadata.page
response.blocks
```

To start asynchronic you can use https://docs.aws.amazon.com/textract/latest/dg/API_StartDocumentTextDetection.html
https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/Textract/Client.html#start_document_text_detection-instance_method
```
document_name = 't.pdf'
bucket = 'duleorlovic-test-us-east-1'
token = SecureRandom.hex
sns_topic_arn = 'arn:aws:sns:us-east-1:219232'
role_arn = 'arn:aws:iam::2192329996'
client = Aws::Textract::Client.new
response = client.start_document_text_detection(
  document_location: { # required
    s3_object: {
      bucket: bucket,
      name: document_name,
    },
  },
  client_request_token: token,
  notification_channel: {
    sns_topic_arn: sns_topic_arn,
    role_arn: role_arn,
  },
)
job_id = response[:job_id]

response = client.get_document_text_detection(job_id: job_id)
```

Data is returned as `Block`, it `:block_type` could be `PAGE`, `LINE`, `WORD`.
also `"KEY_VALUE_SET", "TABLE", "CELL", "SELECTION_ELEMENT"`.
Page block has `[:block_type, :geometry, :id, :relationships]`, Line
has`[:block_type, :confidence, :text, :geometry, :id, :relationships]` and Word
also the same but without relationships.

```
page_block = response.blocks.first
word_block = response.blocks.last

word_block.to_hash.keys
# [:block_type, :confidence, :text, :geometry, :id]
```


# Medical

For Medical https://docs.aws.amazon.com/comprehend/latest/dg/API_medical_DetectEntitiesV2.html or
https://docs.aws.amazon.com/comprehend/latest/dg/API_medical_DetectPHI.html
https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/ComprehendMedical/Client.html#detect_phi-instance_method

```
text = <<TEXT
Clinical Summary Report - 11 Jan 2020 – Boston Hospital
TEXT
client = Aws::ComprehendMedical::Client.new
response = client.detect_phi(text: text)
esponse.entities.map {|p| puts p.inspect }
#<struct Aws::ComprehendMedical::Types::Entity id=0, begin_offset=26, end_offset=37, score=0.9900140762329102, text="11 Jan 2020", category="PROTECTED_HEALTH_INFORMATION", type="DATE", traits=[], attributes=nil>
```
