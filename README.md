# AWS Lambda text_to_speech code

This code uses AWS S3, Poly and Lambda to create a mp3 file from an txt file that is created in an S3 bucket.
You need to set up a S3 trigger to the lambda to work.
It will create it in a output folder with the same name.

```
import json
import boto3
import os
from urllib.parse import unquote_plus

# Initialize the Polly client
polly_client = boto3.client('polly')
s3_client = boto3.client('s3')

def lambda_handler(event, context):
    # Extract bucket name and the file name from the S3 event
    bucket_name = event['Records'][0]['s3']['bucket']['name']
    input_key = unquote_plus(event['Records'][0]['s3']['object']['key'])
    
    # Check if the file is in the 'input' folder and is a .txt file
    if input_key.startswith('input/') and input_key.endswith('.txt'):
        try:
            # Get the content of the .txt file
            response = s3_client.get_object(Bucket=bucket_name, Key=input_key)
            text_content = response['Body'].read().decode('utf-8')
            
            # Call Polly to convert the text to speech
            polly_response = polly_client.synthesize_speech(
                Text=text_content,
                OutputFormat='mp3',
                VoiceId='Joanna'  # You can change this to any voice available in Polly
            )
            
            # Save the MP3 file in the 'output' folder
            output_key = 'output/' + os.path.splitext(os.path.basename(input_key))[0] + '.mp3'
            s3_client.put_object(
                Bucket=bucket_name,
                Key=output_key,
                Body=polly_response['AudioStream'].read(),
                ContentType='audio/mp3'
            )
            
            print(f"MP3 file successfully created and uploaded to {output_key}")
            
            return {
                'statusCode': 200,
                'body': json.dumps(f"MP3 file created at {output_key}")
            }
        
        except Exception as e:
            print(f"Error processing file {input_key}: {e}")
            return {
                'statusCode': 500,
                'body': json.dumps(f"Error processing file: {str(e)}")
            }
    else:
        return {
            'statusCode': 400,
            'body': json.dumps("File is not a .txt file or is not in the 'input' folder")
        }
```
