[<- Back to S3](s3.md)

To implement S3 Pre-signed URLs in a Spring Boot application running on EKS, you’ll follow a flow that leverages the **IRSA (IAM Roles for Service Accounts)** setup you mentioned.

The beauty of this approach is that your Spring Boot application acts as the "Gatekeeper"—it verifies if the user is allowed to see the file, and then gives them a temporary "VIP pass" (the URL) to download it directly from S3.

### The Step-by-Step Workflow

1.  **Request:** The end user (via Browser/Mobile) sends a request to your Spring Boot API (e.g., `GET /api/documents/123/view`).
    
2.  **Authorization:** Your application checks your database to ensure this specific user has permission to view document `123`.
    
3.  **URL Generation:** Your Spring Boot app uses the **AWS SDK** to create a Pre-signed URL.
    
    -   _Note:_ This does **not** involve an actual network call to AWS; the SDK uses the IAM credentials from your Service Account to cryptographically sign a URL locally.
        
4.  **Response:** Your API returns the signed URL string to the user.
    
5.  **Direct Download:** The user’s browser hits the S3 URL directly. S3 verifies the signature and serves the file.
    

### Technical Implementation

#### 1. Add Dependency

In your `pom.xml`, ensure you have the S3 SDK. Since you are using Spring Boot 3, use the **AWS SDK for Java 2.x**:

XML

```
<dependency>
    <groupId>software.amazon.awssdk</groupId>
    <artifactId>s3</artifactId>
</dependency>

```

#### 2. Configure the S3 Presigner Bean

Since you are using IRSA, you don't need to hardcode access keys. The SDK will automatically pick up the credentials from the token file projected into your pod by Kubernetes.

Java

```
@Configuration
public class S3Config {
    @Value("${aws.region}")
    private String region;

    @Bean
    public S3Presigner s3Presigner() {
        return S3Presigner.builder()
                .region(Region.of(region))
                // The DefaultCredentialsProvider handles IRSA automatically
                .build();
    }
}

```

#### 3. Service Logic to Generate the URL

This method creates a link that is valid for a specific duration (e.g., 10 minutes).

Java

```
@Service
public class DocumentService {

    private final S3Presigner presigner;
    private final String bucketName = "your-private-bucket";

    public DocumentService(S3Presigner presigner) {
        this.presigner = presigner;
    }

    public String getPresignedUrl(String objectKey) {
        // Define the request
        GetObjectRequest getObjectRequest = GetObjectRequest.builder()
                .bucket(bucketName)
                .key(objectKey)
                .build();

        // Define the signing options (Expiration)
        GetObjectPresignRequest presignRequest = GetObjectPresignRequest.builder()
                .signatureDuration(Duration.ofMinutes(10))
                .getObjectRequest(getObjectRequest)
                .build();

        // Generate the URL
        PresignedGetObjectRequest presignedRequest = presigner.presignGetObject(presignRequest);
        
        return presignedRequest.url().toString();
    }
}

```

### Why this is the "Better Approach" for you:

-   **Security:** Your S3 bucket remains **100% private**. No public access is ever enabled.
    
-   **Performance:** Your Spring Boot application doesn't "touch" the file bytes. If a user downloads a 1GB file, your application’s memory and CPU usage remain near zero because the data flows directly from S3 to the user.
    
-   **Cost:** You avoid "Data Transfer Out" costs that you would otherwise incur if the data passed through your EKS cluster nodes.
    
-   **Scalability:** Because the signing is a local mathematical operation, your backend can generate thousands of URLs per second without hitting AWS rate limits.


  
In Amazon S3, the `objectKey` (often just called the **Key**) is the **unique identifier or the full path** of the file inside your bucket.

Think of the **Bucket** as the drive letter (like `C:`) and the **Key** as the complete file path after it.

### How S3 "Folders" Work

It is important to know that Amazon S3 is a flat storage system. It doesn't actually have real folders or directories. Instead, it uses the `objectKey` to mimic a folder structure using forward slashes (`/`).

## S3 Object Mapping Examples

| **If your file is located at...** | **Bucket Name** | **Object Key** |
|---|---|---|
| **Directly in the root:** `resume.pdf` | `your-private-bucket` | `resume.pdf` |
| **Inside a user folder:** `users/123/profile.jpg` | `your-private-bucket` | `users/123/profile.jpg` |
| **Deeply nested:** `claims/2026/may/docs/invoice.pdf` | `your-private-bucket` | `claims/2026/may/docs/invoice.pdf` |


### How to use it in your Database Design

When an end user uploads a document, your Spring Boot application will typically do two things:

1.  Upload the file to S3 using a unique key.
    
2.  Save that exact `objectKey` string into your database table (e.g., a `documents` table).
    

When a user later wants to view that document, your API will look up the record in the database, grab the `objectKey`, and pass it to your `getPresignedUrl(objectKey)` method to generate the link.

> **Tip:** Always include the full path starting _after_ the bucket name, and do not include a leading slash (use `images/pic.png`, not `/images/pic.png`).
