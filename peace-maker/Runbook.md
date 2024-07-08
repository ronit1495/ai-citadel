### Scenario: User Data Sync Error for APAC Region

**APAC region uses AWS CloudFront.**

### Runbook for Handling the Errors

#### MongoDB Connection Error
1. **Alert**: `MongoDB connection failed.`
   - **Steps**:
     1. Verify MongoDB connection string and ensure it is correct.
     2. Check if the MongoDB secrets were rotated recently. Update the credentials in the peace-keeper configuration accordingly.
     3. Test the MongoDB connection with the updated credentials.
     4. Restart the peace-keeper container to apply the new credentials.
     5. Monitor the logs to ensure successful connection to MongoDB.

#### Data Retrieval Error
1. **Alert**: `Failed to retrieve invalidation results from MongoDB.`
   - **Steps**:
     1. Verify that MongoDB is accessible with the new credentials.
     2. Check the MongoDB collection `invalidation_results` for any data consistency issues.
     3. Ensure the application has the necessary read permissions for the `invalidation_results` collection.
     4. Retry the data retrieval process after fixing the connection issues.

#### AWS CloudFront Invalidation Request Error
1. **Alert**: `CloudFront invalidation request failed.`
   - **Steps**:
     1. Identify the missing invalidation results caused by MongoDB retrieval failure.
     2. Manually trigger the invalidation request using the correct distribution ID and paths.
     3. Verify the success of the invalidation request through the AWS CloudFront console.
     4. Ensure that future invalidation requests can retrieve the necessary data from MongoDB.

#### User Impact Notification
1. **Alert**: `User data Sync error for APAC region.`
   - **Steps**:
     1. Communicate with the affected users about the data sync issue and potential impact.
     2. Provide an estimated timeline for resolution and any workaround if available.
     3. Monitor the system for further errors and ensure the issue is resolved promptly.
     4. Document the incident and update the runbook with any new steps taken to resolve the issue.
