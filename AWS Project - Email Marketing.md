# Email Marketing Campaign System with AWS

This README provides code for AWS Lambda FUnction to send an email using AWS services including SES, and EventBridge. It also includes a policy to grant the necessary permissions to the Lambda function.

## Lambda Function Python Code

The Lambda function is responsible for reading contacts from an S3 bucket, personalizing email templates, and sending emails using SES.

### Python Code

```python
import boto3
import csv

# Initialize the boto3 client for S3 and SES
s3_client = boto3.client('s3')
ses_client = boto3.client('ses')

def lambda_handler(event, context):
    # Specify the S3 bucket name
    bucket_name = 'ttt-email-marketing'  # Replace with your bucket name

    try:
        # Retrieve the CSV file containing contacts from S3
        csv_file = s3_client.get_object(Bucket=bucket_name, Key='contacts.csv')
        lines = csv_file['Body'].read().decode('utf-8').splitlines()
        
        # Retrieve the HTML email template from S3
        email_template = s3_client.get_object(Bucket=bucket_name, Key='email_template.html')
        email_html = email_template['Body'].read().decode('utf-8')
        
        # Parse the CSV file
        contacts = csv.DictReader(lines)
        
        for contact in contacts:
            # Replace placeholders in the email template with contact information
            personalized_email = email_html.replace('{{FirstName}}', contact['FirstName'])
            
            # Send the email using SES
            response = ses_client.send_email(
                Source='you@yourdomainname.com',  # Replace with your verified "From" address
                Destination={'ToAddresses': [contact['Email']]},
                Message={
                    'Subject': {'Data': 'Your Weekly Tiny Tales Mail!', 'Charset': 'UTF-8'},
                    'Body': {'Html': {'Data': personalized_email, 'Charset': 'UTF-8'}}
                }
            )
            print(f"Email sent to {contact['Email']}: Response {response}")
    except Exception as e:
        print(f"An error occurred: {e}")
```

#### Explanation

- **Import Boto3 and CSV libraries**: These libraries are used to interact with AWS services and read CSV files.
- **Initialize Boto3 clients**: Create clients for S3 and SES to interact with these services.
- **Lambda Handler Function**: This function is triggered by an event (like a scheduled EventBridge rule). 
  - **Retrieve CSV File**: Fetch the `contacts.csv` file from S3.
  - **Retrieve Email Template**: Fetch the `email_template.html` file from S3.
  - **Parse CSV File**: Read and parse the CSV file containing contact details.
  - **Personalize and Send Emails**: Loop through each contact, personalize the email template, and send the email using SES.

## Lambda Function Test Event

Use the following JSON as a test event to simulate a scheduled execution of the Lambda function. The function does not use this event data but requires an event to trigger.

```json
{
  "comment": "Generic test event for scheduled Lambda execution. The function does not use this event data.",
  "test": true
}
```

## IAM Policy for SES and S3 Permissions

The Lambda function requires permissions to access S3 and SES. The following IAM policy grants the necessary permissions.

### Policy JSON

Update the ARN to use your S3 bucket:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject"
            ],
            "Resource": "arn:aws:s3:::ttt-email-marketing/*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "ses:SendEmail",
                "ses:SendRawEmail"
            ],
            "Resource": "*"
        }
    ]
}
```

#### Explanation

- **s3:GetObject**: Allows the function to read objects (files) from the specified S3 bucket.
- **ses:SendEmail and ses:SendRawEmail**: Allow the function to send emails using SES.

## Conclusion

By following this guide, you will have a fully functional email marketing campaign system leveraging AWS services. Ensure all required AWS resources (S3 bucket, SES setup, IAM roles) are correctly configured and permissions are properly set.

If you encounter any issues or have questions, feel free to ask for help.

---