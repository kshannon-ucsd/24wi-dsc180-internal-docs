# End-to-End Summary: Solving API Deployment with Subdomains & HTTPS

This process involved deploying a machine learning API, securing it with HTTPS, and connecting it to a React frontend. Below is a step-by-step summary of how we solved the problem.

---

## ✅ Step 1: Creating Subdomains in Route 53
### **Problem:**
- A dedicated subdomain was needed for the API.
- The ALB DNS name was not user-friendly and lacked HTTPS.

### **Solution:**
- Created a **CNAME record** in Route 53 to point the custom subdomain to the ALB.
- This allowed users to access the API via a user-friendly domain.

✅ **Now, the API is accessible via a custom domain.**

---

## ✅ Step 2: Configuring ALB to Route API Traffic
### **Problem:**
- The ALB needed to route requests to the correct ECS service.

### **Solution:**
- Added an **ALB Listener Rule** to check if the Host Header matches the custom subdomain.
- If matched, ALB forwarded the request to the correct ECS **Target Group** running the API.

✅ **Now, API requests are correctly routed to the backend.**

---

## ✅ Step 3: Securing API with HTTPS (AWS ACM)
### **Problem:**
- Browsers block insecure (`http://`) requests due to missing SSL certificates.
- AWS ALB requires a valid **SSL certificate** to serve HTTPS requests.

### **Solution:**
- Requested a **public SSL certificate** in **AWS Certificate Manager (ACM)** for the API subdomain.
- Validated domain ownership via **CNAME records** in Route 53.
- Once validated, attached the SSL certificate to the **ALB HTTPS (443) listener**.

✅ **Now, the API is accessible securely over HTTPS.**

---

## ✅ Step 4: Connecting React Frontend to API
### **Problem:**
- The React app needed to send images to the API and display predictions.
- Initial API calls failed due to incorrect URL handling and CORS issues.

### **Solution:**
- Updated **API_URL** in React to use `https://<subdomain>/predict`.
- Allowed users to upload images and automatically test with a sample file.
- Ensured API requests sent **FormData** properly for image uploads.

✅ **Now, the frontend correctly interacts with the API and displays predictions.**

---

## ✅ Step 5: Fixing API Response Parsing in React
### **Problem:**
- The API response was wrapped in an extra **"body"** field, causing JSON parsing issues.
- The frontend wasn't correctly extracting the prediction value.

### **Solution:**
- Updated Flask API to return direct JSON (`{ "prediction": 1 }`).
- Updated React code to correctly parse `data.body` if backend changes were not possible.

✅ **Now, React successfully displays predictions received from the API.**

---

## 🔥 Final Summary: How We Solved Everything

| **Step** | **Problem** | **Solution** | **Result** |
|---------|------------|-------------|------------|
| 1️⃣ Create Subdomains | ALB DNS is not user-friendly | Created a custom subdomain in Route 53 | Requests now use a clean domain |
| 2️⃣ Configure ALB Routing | ALB does not know where to send traffic | Added Host Header rules to ALB | ALB correctly routes API requests |
| 3️⃣ Enable HTTPS with ACM | API requests blocked due to insecure HTTP | Issued SSL certificate via ACM | API is now secure with HTTPS |
| 4️⃣ Connect React to API | Frontend couldn't send images to API | Updated API URL & sent images via FormData | React app uploads images & gets predictions |
| 5️⃣ Fix API Response Parsing | API response was wrapped in "body" field | Fixed JSON parsing in React | React correctly displays predictions |

---
