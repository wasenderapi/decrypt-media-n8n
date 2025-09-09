# WAsender API Media Decryption with n8n

This guide demonstrates how to integrate the WAsender API's decrypt-media-file endpoint with n8n workflows to decrypt WhatsApp media files.

## Overview

The WAsender API provides a media decryption service that allows you to decrypt encrypted WhatsApp media files. This integration enables you to use this functionality within your n8n automation workflows.

## Prerequisites

- n8n instance (self-hosted or cloud)
- WAsender API account and API key
- Basic understanding of n8n workflows
- Encrypted media files from WhatsApp

## API Endpoint

```
POST https://wasenderapi.com/api-docs/messages/decrypt-media-file
```

## Authentication

The WAsender API uses API key authentication. You'll need to:

1. Sign up for a WAsender API account
2. Obtain your API key from the dashboard
3. Configure the API key in your n8n workflow

## Setup Instructions

### Step 1: Create HTTP Request Node

1. In your n8n workflow, add an **HTTP Request** node
2. Configure the node with the following settings:

**Basic Settings:**
- **Method**: `POST`
- **URL**: `https://wasenderapi.com/api-docs/messages/decrypt-media-file`

**Authentication:**
- **Authentication**: `Generic Credential Type`
- **Generic Auth Type**: `Header Auth`
- **Name**: `Authorization`
- **Value**: `Bearer YOUR_API_KEY`

### Step 2: Configure Request Body

In the **Body** section of the HTTP Request node:

**Content Type**: `application/json`

**Body (JSON):**
```json
{
  "encryptedMedia": "{{ $json.encryptedMediaData }}",
  "mediaKey": "{{ $json.mediaKey }}",
  "mediaType": "{{ $json.mediaType }}"
}
```

### Step 3: Handle Response

The API will return decrypted media data. Configure your workflow to handle the response:

```json
{
  "success": true,
  "decryptedMedia": "base64_encoded_media_data",
  "mediaType": "image/jpeg",
  "fileSize": 156789
}
```

## Complete n8n Workflow Example

Here's a complete workflow configuration:

### Workflow JSON
```json
{
  "nodes": [
    {
      "parameters": {},
      "name": "When clicking \"Execute\"",
      "type": "n8n-nodes-base.manualTrigger",
      "typeVersion": 1,
      "position": [460, 540]
    },
    {
      "parameters": {
        "httpMethod": "POST",
        "url": "https://wasenderapi.com/api-docs/messages/decrypt-media-file",
        "authentication": "genericCredentialType",
        "genericAuthType": "httpHeaderAuth",
        "options": {},
        "bodyParametersUi": {
          "parameter": [
            {
              "name": "encryptedMedia",
              "value": "={{ $json.encryptedMediaData }}"
            },
            {
              "name": "mediaKey", 
              "value": "={{ $json.mediaKey }}"
            },
            {
              "name": "mediaType",
              "value": "={{ $json.mediaType }}"
            }
          ]
        }
      },
      "name": "Decrypt Media",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.1,
      "position": [680, 540],
      "credentials": {
        "httpHeaderAuth": {
          "id": "wasender_api_key",
          "name": "WAsender API Key"
        }
      }
    }
  ],
  "connections": {
    "When clicking \"Execute\"": {
      "main": [
        [
          {
            "node": "Decrypt Media",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  }
}
```

## Input Data Format

Your workflow should provide the following input data:

```json
{
  "encryptedMediaData": "base64_encoded_encrypted_media",
  "mediaKey": "media_decryption_key", 
  "mediaType": "image/jpeg"
}
```

### Supported Media Types
- `image/jpeg`
- `image/png`
- `video/mp4`
- `audio/mp3`
- `audio/ogg`
- `application/pdf`

## Error Handling

Add error handling to your workflow to manage API failures:

### Common Error Responses

**Invalid API Key (401):**
```json
{
  "error": "Unauthorized",
  "message": "Invalid API key"
}
```

**Invalid Media Data (400):**
```json
{
  "error": "Bad Request", 
  "message": "Invalid encrypted media data"
}
```

**Rate Limit Exceeded (429):**
```json
{
  "error": "Too Many Requests",
  "message": "Rate limit exceeded"
}
```

### Error Handling Node Configuration

Add an **IF** node after the HTTP Request to check for errors:

**Condition**: `{{ $json.success }} === true`

- **True**: Continue with decrypted media processing
- **False**: Handle error (log, notify, retry, etc.)

## Usage Examples

### Example 1: Decrypt WhatsApp Image

**Input:**
```json
{
  "encryptedMediaData": "iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mP8/5+hHgAHggJ/PchI7wAAAABJRU5ErkJggg==",
  "mediaKey": "abc123def456789",
  "mediaType": "image/jpeg"
}
```

**Expected Output:**
```json
{
  "success": true,
  "decryptedMedia": "base64_decrypted_image_data",
  "mediaType": "image/jpeg",
  "fileSize": 2048
}
```

### Example 2: Decrypt WhatsApp Video

**Input:**
```json
{
  "encryptedMediaData": "encrypted_video_base64_data",
  "mediaKey": "video_decryption_key_123",
  "mediaType": "video/mp4"
}
```

## Best Practices

1. **Secure API Key Storage**: Store your API key in n8n credentials, never hardcode it
2. **Error Handling**: Always implement proper error handling for API calls
3. **Rate Limiting**: Be aware of API rate limits and implement appropriate delays
4. **Data Validation**: Validate input data before sending to the API
5. **Logging**: Log important events for debugging and monitoring

## Troubleshooting

### Common Issues

**Issue**: "Invalid API key" error
**Solution**: Verify your API key is correct and has proper permissions

**Issue**: "Invalid media data" error  
**Solution**: Ensure the encrypted media data is properly base64 encoded

**Issue**: "Media key not found" error
**Solution**: Verify the media key corresponds to the encrypted media file

**Issue**: Network timeout
**Solution**: Increase timeout settings in the HTTP Request node

### Debug Mode

Enable debug mode in n8n to see detailed request/response data:

1. Go to Settings → Debug
2. Enable "Save manual executions"
3. Check execution logs for detailed error information

## Security Considerations

- Never expose your API key in workflow exports
- Use environment variables or n8n credentials for sensitive data
- Validate and sanitize all input data
- Implement proper access controls for workflows containing sensitive operations

## Support

For issues related to:
- **n8n functionality**: Check the [n8n documentation](https://docs.n8n.io/)
- **WAsender API**: Contact WAsender support or check their API documentation
- **This integration**: Open an issue in this repository

## License

This documentation is provided as-is for educational purposes.