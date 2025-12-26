LAMBDA Function Python 3.14

LAMBDA PRO (final code for phase 1 ):

import json
import boto3
from botocore.exceptions import ClientError

# 1. Initialize Clients for Mumbai Region
dynamodb = boto3.resource('dynamodb', region_name='ap-south-1')
ses = boto3.client('ses', region_name='ap-south-1')

# Reference your table name exactly
table = dynamodb.Table('UserSubmissions')

def lambda_handler(event, context):
    # 2. Extract parameters from the GET request URL
    params = event.get('queryStringParameters', {})
    user_name = params.get('name', 'Tony')
    user_email = params.get('email', 'your-verified-mail@gmail.com')

    # Your verified email address
    MY_VERIFIED_EMAIL = "your-verified-mai@gmail.com" 

    try:
        # 3. Step A: Insert data into DynamoDB
        table.put_item(
            Item={
                'email': user_email,
                'name': user_name
            }
        )

        # 4. Step B: Send confirmation email via SES
        # Source must be your verified email
        ses.send_email(
            Source=MY_VERIFIED_EMAIL,
            Destination={'ToAddresses': [user_email]},
            Message={
                'Subject': {'Data': 'Form Submission Success!'},
                'Body': {
                    'Text': {'Data': f"Hello {user_name},\n\nYour data has been successfully saved to the UserSubmissions table in Mumbai (ap-south-1)."}
                }
            }
        )

        # 5. Final Success Return
        return {
    'statusCode': 200,
    'headers': {
        'Access-Control-Allow-Origin': '*', # This must match your API Gateway
        'Access-Control-Allow-Headers': 'Content-Type',
        'Access-Control-Allow-Methods': 'GET,OPTIONS'
    },
    'body': json.dumps("Success! Data saved and Email sent.")
}

    except ClientError as e:
        return {
            'statusCode': 500,
            'body': json.dumps(f"AWS Error: {e.response['Error']['Message']}")
        }
    except Exception as e:
        return {
            'statusCode': 500,
            'body': json.dumps(f"General Error: {str(e)}")
        }
