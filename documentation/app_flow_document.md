# Laundry Management App Flow Document

## Onboarding and Sign-In/Sign-Up

When a new user first visits the application, they land on the public home page where they can choose to check laundry status or sign in to their account. If the user does not have an account yet, they click on the Sign Up link which opens a registration form. In this form, they enter their email address, create a password, and confirm their role as either an employee (Pegawai) or owner. After submitting the form, they receive a confirmation email. Clicking the verification link in that email activates their account and redirects them to the Sign In page.

To sign in, the user navigates to the Sign In page and enters their registered email and password. The system uses NextAuth for authentication and issues a session token upon successful login. If the credentials are incorrect, the page displays an error message prompting the user to try again. A Forgot Password link on the sign-in form leads to a password recovery page where the user submits their email address. The system sends a password reset link by email, and clicking that link opens a page to enter and confirm a new password. After resetting the password, the user is redirected back to the Sign In page to log in with the new credentials. Once signed in, the user remains logged in until they choose to sign out, which is accessible via a Logout button in the header on any protected page.

## Main Dashboard or Home Page

After a successful login, the user lands on a role-specific dashboard. The layout features a fixed sidebar on the left, a header across the top, and a main content area. The header shows the application name, the user’s role, and a profile avatar with a dropdown menu containing links to account settings and logout. The sidebar presents menu items based on the user’s role. An employee sees entries labeled Order Entry, Customer Data, and Transaction List. An owner sees Employee Management, Reports, and System Monitoring. Clicking any sidebar item updates the main content area without a full page reload, keeping the header and sidebar in place. This layout ensures the user always has quick access to navigation and account controls.

## Detailed Feature Flows and Page Transitions

### Public Laundry Status Check Flow

A public user who is not signed in can visit the Home page and click the Check Status link. They arrive at a status lookup page with a search form asking for an order number or phone number. After entering the required information and submitting, the page displays a list of matching orders inside cards. Each card shows the customer name, order date, service type, current status, and expected completion date. If no records are found, a friendly message explains that no matching orders exist. The user can refine their search or return to the Home page at any time.

### Employee (Pegawai) Workflow

When an employee selects Order Entry from the sidebar, the main area displays a form to create a new laundry order. The employee enters customer details, selects service type from a dropdown, inputs weight, and picks a due date. Clicking the Save button triggers a server-side action that validates the input. If any field is invalid or missing, an inline error message appears above the form field. Upon successful save, a toast notification confirms that the order was created and the form resets for a new entry.

From the sidebar, choosing Customer Data brings up a table listing all customers. The table includes columns for name, phone number, and total orders. The employee can click any row to open a detail view where they can edit contact information. Submitting changes updates the database via a server action and shows a success notification.

Selecting Transaction List shows a paginated table of all orders with filters for date range and status. An Update Status button on each row opens a dialog where the employee can change the status to In Progress, Completed, or Picked Up. Confirming the dialog triggers a server action to update the order and reloads the table data automatically.

### Owner Workflow

An owner clicking on Employee Management sees a table of all employees. This table displays name, role, and hire date. The owner can click Add Employee to open a form where they enter a new employee’s name, email, role, and temporary password. Saving the form creates the account and sends an invitation email with instructions to set a permanent password. Owner can also edit or deactivate existing employees by opening an edit form, making changes, and saving.

In the Reports section, the owner finds financial and operational dashboards. The page loads charts and tables showing metrics such as monthly revenue, total orders, and average processing time. Owners can adjust date filters at the top and the data refreshes accordingly. A Print Report button generates a PDF version of the current view.

The System Monitoring page displays real-time statistics on current orders, machine usage, and staff activity. Data updates every minute via a polling mechanism. If the owner needs to jump back to any other section, they simply click the corresponding item in the sidebar.

## Settings and Account Management

Any user can access their account settings by clicking their profile avatar in the header. The account settings page shows personal information fields such as name, email, and phone number. Changing any field requires entering the current password to confirm identity. A separate section on the same page allows the user to change their password by providing the current password, a new password, and password confirmation. Saving either form triggers a server action and displays a toast indicating success or describing validation errors. After updating settings, the user clicks Back to Dashboard in the header to return to their main view.

## Error States and Alternate Paths

If a user tries to navigate to a protected page without being signed in, they are redirected automatically to the Sign In page. When the application loses network connectivity while a user is interacting, an offline banner appears at the top of the page. Any action that requires server communication will show a spinner and then an error notification if it fails. In forms, invalid input fields are highlighted with messages explaining the problem. For pages or records that do not exist, a friendly 404 page appears with a link back to the dashboard or home page. Whenever an error occurs, the user can retry the action or navigate elsewhere without breaking the overall layout.

## Conclusion and Overall App Journey

From the very first visit, a user can explore the public status lookup or create an account. Once registered and signed in, they enter a personalized dashboard that adapts to their role. Employees can quickly enter new orders, manage customer data, and update order status. Owners can oversee employees, review detailed reports, and monitor real-time operations. At any point, users can update their profile details, change their password, or sign out. Error states guide users back to a stable flow, ensuring that every action leads to an expected next step. This journey, from sign-up to day-to-day management, provides a cohesive and intuitive experience for both front-end customers and internal staff.