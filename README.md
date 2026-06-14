# SettleUp - Expense Sharing Web App (Splitwise Clone)
SettleUp is a full-stack web application designed to help users track shared expenses, balances, and settle debts within friends and groups. 
**Tech Stack:**
*   **Backend:** Java, Spring Boot, Spring Security, Hibernate/JPA, PostgreSQL/MySQL
*   **Frontend:** React, React Router, Tailwind CSS, Axios
*   **Authentication:** JWT (JSON Web Tokens)
---
## Phase 1: System Design & Understanding Core Features
Before coding, it's crucial to understand the core database entities and relationships:
1.  **User:** Has an ID, Name, Email, Password.
2.  **Group:** Has an ID, Name, Description. A Group can have multiple Users.
3.  **Expense:** Has an ID, Description, Total Amount, Date, Group/Payer, and Split Type (Equal, Exact, Percentage).
4.  **ExpenseShare (Owed/Paid):** Links an Expense to a User. Records how much that specific user paid and how much they actually owe.
**Core Algorithm:** "Simplifying Debts" (Network Flow / Greedy Algorithm) 
You need logic to turn `A owes B $10` and `B owes C $10` into just `A owes C $10`.
---
## Phase 2: Backend Development (Spring Boot)
### Step 1: Initialize the Project
1. Go to [Spring Initializr](https://start.spring.io/).
2. Select **Maven**, **Java**, and the latest stable Spring Boot version.
3. Add Dependencies: 
   *   Spring Web
   *   Spring Data JPA
   *   PostgreSQL Driver (or MySQL)
   *   Spring Security
   *   Lombok
   *   Validation
4. Download, extract, and open the project in your IDE.
### Step 2: Database Configuration
In src/main/resources/application.properties (or .yml), configure your database
### Step 3: Create JPA Entities
Create your domain models in a models or entities package:
*   User.java (implementing UserDetails for Security)
*   Group.java (ManyToMany relationship with Users)
*   Expense.java (ManyToOne with Group)
*   UserShare.java (Matches a User to an Expense, detailing mountPaid and mountOwed)
### Step 4: Repositories and Services
1.  Create standard Spring Data JPA Repositories for the entities.
2.  **UserService:** Handle user registration, password hashing (BCrypt), and profile fetching.
3.  **GroupService:** Handle creating groups and adding members.
4.  **ExpenseService:** Calculate splits. For example, if `` split equally among 4 people, generate 4 UserShare records with `` owed each.
5.  **SettlementService:** The core module. Fetch all expenses for a group, calculate the net balance for each user (Total Paid - Total Owed), and generate settlement transactions (Who pays whom).
### Step 5: REST Controllers
Create REST APIs to expose to the frontend:
*   POST /api/auth/register & POST /api/auth/login
*   GET /api/groups, POST /api/groups/{id}/members
*   POST /api/expenses (Send payload with total, payer, and list of participants to split among)
*   GET /api/groups/{groupId}/balances (Returns simplified debts)
### Step 6: Security & JWT
1.  Create a JwtUtil class to generate and validate tokens.
2.  Configure a SecurityFilterChain to allow unauthenticated access to /api/auth/** but require authentication via a JWT Filter for everything else.
---
## Phase 3: Frontend Development (React)
### Step 1: Initialize the Project
Using Vite is recommended for modern React apps. Open your terminal:
``bash
npm create vite@latest frontend -- --template react
cd frontend
npm install
``
### Step 2: Install Dependencies
``bash
npm install react-router-dom axios
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
``
Configure 	ailwind.config.js to scan your React files and add the Tailwind directives to index.css.
### Step 3: Project Structure
Organize your src folder:
*   /components: Reusable UI elements (Navbar, Button, Modal).
*   /pages: Route components (Login, Register, Dashboard, GroupDetail).
*   /services: Axios API caller configurations and route endpoints.
*   /context: React Context for Global State (Auth state).
### Step 4: Routing Setup
In App.jsx, configure your routes using React Router:
*   /login: User authentication.
*   /register: New account creation.
*   /dashboard: View total balances, list of groups, and recent activity.
*   /group/:id: View group members, list expenses, settle debts.
### Step 5: API Integration with Axios
Create an api.js file:
``javascript
import axios from 'axios';
const api = axios.create({
    baseURL: 'http://localhost:8080/api',
});
// Add a specific interceptor to attach the JWT token to every request
api.interceptors.request.use((config) => {
    const token = localStorage.getItem('token');
    if (token) {
        config.headers.Authorization = "Bearer " + token;
    }
    return config;
});
export default api;
``
### Step 6: Build the Features
1.  **Auth Flow:** Make login request -> save JWT to localStorage -> redirect to Dashboard.
2.  **Dashboard:** Fetch groups from backend and map over them to create Group Cards.
3.  **Add Expense Modal:** A form where users choose a Group, input an Amount, Description, and select split logic. Send this back to POST /api/expenses.
4.  **Balances UI:** Display exactly who owes whom in a clean list format utilizing the data from the SettlementService.
---
## Phase 4: Running the App
**Backend:**
1. Ensure your Postgres/MySQL server is running.
2. In your Spring Boot root directory, run:
   ``bash
   mvn spring-boot:run

---
## Future Enhancements
*   Add different split types (by percentage, by exact amounts).
*   Add OAuth2 (Google/Facebook Login).
*   Upload bills/receipts securely using AWS S3.
*   Mobile responsive design for easy usage on phones.
