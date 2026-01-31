# Home

## Observe Portal

### Overview

A standalone observability interface that can be deployed by Organisations who already have metrics available and need multi-tenant visualization, and administrative control over dashboards and access.

### What You'll Get



* **Grafana Integration**: Centralized dashboard, folder, team, and user management
* **Organization Management**: Multi-tenant support for different organizations
* **Team Management**: Organize users into teams with specific access levels
* **Secure Authentication**: Adopter-integrated authentication with captcha and JWT tokens
* **Modern Interface**: Clean, responsive UI built with Next.js and Tailwind CSS

### Architecture

The portal consists of two main components:

1. **Frontend (Next.js)**: Web interface for portal administration
2. **Backend (FastAPI)**: Authentication service with PostgreSQL database

### Prerequisites

Before you begin, ensure you have the following installed:

#### System Requirements

* **Node.js**: Version 16.x or higher
* **Python**: Version 3.8 or higher
* **PostgreSQL**: Version 12 or higher
* **Grafana**: Version 8.x or higher (running instance)
* **Git**: For cloning the repository

#### Required Access

* Admin access to a running Grafana instance
* PostgreSQL database with create privileges
* Email service credentials (for user notifications)

### Project Structure

```
AI4Voice_Portal/
├── app/                    # Next.js Frontend Application
├── components/             # React Components
├── hooks/                  # Custom React Hooks
├── lib/                    # Frontend Utilities & API Clients
├── types/                  # TypeScript Type Definitions
├── backend/                # Backend Services
│   └── app/                # FastAPI Authentication Service
├── package.json            # Frontend Dependencies
├── next.config.ts          # Next.js Configuration
└── README.md               # This file
```

***

### Quick Start Installation

#### Step 1: Clone the Repository

```
git clone https://github.com/COSS-India/observe.git
cd observe
```

#### Step 2: Set Up the Backend (Authentication Service)

**2.1 Navigate to Backend Directory**

```
cd backend/app
```

**2.2 Create Python Virtual Environment**

**On Linux/macOS:**

```
python -m venv venv
source venv/bin/activate
```

**On Windows:**

```
python -m venv venv
venv\Scripts\activate
```

**2.3 Install Python Dependencies**

```
pip install -r requirements.txt
```

**2.4 Configure Environment Variables**

Create a `.env` file in the `backend/app` directory:

```
cp env.example .env
```

Edit the `.env` file with your configuration:

```
# Database Configuration
DATABASE_URL=postgresql://username:password@localhost:5432/observe_db

# JWT Configuration
SECRET_KEY=your-super-secret-key-change-this
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30

# Email Configuration
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@example.com
SMTP_PASSWORD=your-email-password

# Application Settings
DEBUG=False
ALLOWED_ORIGINS=http://localhost:3000
```

**Important**: Replace all placeholder values with your actual credentials.

**2.5 Initialize the Database**

```
python init_db.py
```

This creates the necessary database tables and schema.

**2.6 Start the Backend Server**

```
python -m uvicorn app.main:app --reload --port 8000
```

The backend API will be available at `http://localhost:8000`

You can view the API documentation at `http://localhost:8000/docs`

#### Step 3: Set Up the Frontend (Portal Interface)

**3.1 Return to Root Directory**

```
cd ../..  # Back to the observe root directory
```

**3.2 Install Node Dependencies**

```
npm install
```

**3.3 Configure Environment Variables**

Create a `.env.local` file in the root directory:

```
touch .env.local
```

Add the following configuration:

```
# Backend API URL
BACKEND_URL=http://localhost:8000

# Grafana Configuration
NEXT_PUBLIC_GRAFANA_URL=http://localhost:3001
GRAFANA_API_KEY=your_grafana_api_key_here

# NextAuth Configuration
NEXTAUTH_SECRET=generate-a-random-secret-here
NEXTAUTH_URL=http://localhost:3000
```

**How to get Grafana API Key:**

1. Log in to your Grafana instance
2. Go to Configuration → API Keys
3. Click "New API Key"
4. Set name as "Observe Portal" with Admin role
5. Copy the generated key

**Generate NEXTAUTH\_SECRET:**

```
openssl rand -base64 32
```

**3.4 Start the Development Server**

```
npm run dev
```

The frontend will be available at `http://localhost:3000`

#### Step 4: Verify Installation

1. **Check Backend**: Visit `http://localhost:8000/docs` - you should see the API documentation
2. **Check Frontend**: Visit `http://localhost:3000` - you should see the login page
3. **Check Grafana**: Ensure your Grafana instance is running at the configured URL

***

### First Time Setup

#### Create Admin User



1. Access the frontend at `http://localhost:3000`
2. Click "Sign Up" to create your first admin account
3. Complete the captcha verification
4. Verify your email address
5. Log in with your credentials

#### Connect to Grafana

The platform automatically connects to Grafana using the API key you configured. You should now be able to:

* View existing Grafana organizations
* Manage users and teams
* Create and organize dashboards
* Set up role-based access controls

***

### Production Deployment

#### Backend Deployment

**Option 1: Railway**

1. Push your code to GitHub
2. Connect your repository to Railway
3. Add environment variables in Railway dashboard
4. Deploy

**Option 2: AWS/DigitalOcean**

1. Set up a server with Python 3.8+
2. Clone the repository
3. Install dependencies
4. Configure environment variables
5. Use gunicorn or uvicorn with systemd service:

```
uvicorn app.main:app --host 0.0.0.0 --port 8000 --workers 4
```

**Option 3: Docker (Recommended)**

Create `Dockerfile` in `backend/app`:

```
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

Build and run:

```
docker build -t observe-backend .
docker run -p 8000:8000 --env-file .env observe-backend
```

#### Frontend Deployment

**Option 1: Vercel (Recommended)**

1. Push your code to GitHub
2. Import project in Vercel
3. Add environment variables:
   * `BACKEND_URL`: Your production backend URL
   * `NEXT_PUBLIC_GRAFANA_URL`: Your Grafana URL
   * `GRAFANA_API_KEY`: Your Grafana API key
   * `NEXTAUTH_SECRET`: Your secret
   * `NEXTAUTH_URL`: Your production URL
4. Deploy

**Option 2: Netlify**

Similar to Vercel - import repository and set environment variables.

**Option 3: Self-hosted**

```
npm run build
npm start
```

Use nginx as reverse proxy:

```
server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

#### Database Setup for Production

Use a managed PostgreSQL service:

* **AWS RDS**: For enterprise deployments
* **DigitalOcean Managed Databases**: Cost-effective option
* **Heroku Postgres**: Easy setup
* **Supabase**: Modern alternative with additional features

Update `DATABASE_URL` in your backend `.env` file with the production database connection string.

### Configuration Guide

#### Grafana Integration

1. **API Key Permissions**: Ensure your Grafana API key has Admin role
2. **CORS Settings**: Configure Grafana to allow requests from your frontend URL
3. **Organizations**: Set up organizations in Grafana before managing them in Observe

#### Email Service

The platform supports multiple email providers:

**Gmail:**

```
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASSWORD=app-specific-password
```

**SendGrid:**

```
SMTP_HOST=smtp.sendgrid.net
SMTP_PORT=587
SMTP_USER=apikey
SMTP_PASSWORD=your-sendgrid-api-key
```

**AWS SES:**

```
SMTP_HOST=email-smtp.us-east-1.amazonaws.com
SMTP_PORT=587
SMTP_USER=your-smtp-username
SMTP_PASSWORD=your-smtp-password
```

#### Security Best Practices



1. **Change Default Secrets**: Always generate new `SECRET_KEY` and `NEXTAUTH_SECRET`
2. **Use HTTPS**: Enable SSL/TLS in production
3. **Environment Variables**: Never commit `.env` files to version control
4. **Database**: Use strong passwords and restrict access
5. **API Keys**: Rotate Grafana API keys regularly
6. **CORS**: Configure allowed origins appropriately

***

### Troubleshooting

#### Backend Issues



**Database connection error:**

* Verify PostgreSQL is running
* Check `DATABASE_URL` format: `postgresql://user:pass@host:port/dbname`
* Ensure database exists and user has permissions

**Port already in use:**

```
# Find process using port 8000
lsof -i :8000
# Kill the process
kill -9 <PID>
```

#### Frontend Issues



**Environment variables not loading:**

* Ensure `.env.local` is in the root directory
* Restart the development server after changes
* Check that variable names start with `NEXT_PUBLIC_` for client-side access

**Grafana connection failed:**

* Verify Grafana URL is accessible
* Check API key validity in Grafana settings
* Ensure CORS is configured in Grafana

#### Common Errors



**"Module not found" errors:**

```
# Delete node_modules and reinstall
rm -rf node_modules
npm install
```

**Database migration issues:**

```
# Reset database (WARNING: This deletes all data)
python init_db.py --reset
```

***

### Maintenance

#### Regular Updates



```
# Update backend dependencies
cd backend/app
pip install -r requirements.txt --upgrade

# Update frontend dependencies
cd ../..
npm update
```

#### Database Backups



```
# Backup PostgreSQL database
pg_dump -U username -d observe_db > backup_$(date +%Y%m%d).sql

# Restore from backup
psql -U username -d observe_db < backup_20240101.sql
```

#### Log Management



Backend logs location: `backend/app/logs/`

Monitor logs in production:

```
tail -f logs/app.log
```

***

### Support and Community



* **GitHub Issues**: Report bugs or request features at [https://github.com/COSS-India/observe/issues](https://github.com/COSS-India/observe/issues)
* **Documentation**: Check the repository README for updates
* **Contributing**: See `CONTRIBUTING.md` in the repository

***

### Video Tutorials

A short video walkthrough of the Observe Building Blocks and its key functionalities are available here :

[Observe Building Block v1.0](https://www.youtube.com/playlist?list=PLnvlKntxa4FpHX6yIqr4hpFm-wc5qHUPP)

[ Observe Building block v1.1](https://youtu.be/sABEDKOrO-Q)

### Next Steps

After successful setup:

1. **User Onboarding**: Create user accounts for your team
2. **Dashboard Configuration**: Import or create Grafana dashboards
3. **Team Setup**: Organize users into teams with appropriate access
4. **Monitoring**: Set up monitoring for the platform itself
5. **Backup Strategy**: Implement regular database backups

### Team and Dashboard Provisioning By Super Admin



#### Team Creation

Super Admins can create teams and map them to specific organizations. This allows for better organization and management of users within the system.

* Navigate to the **Team Management** page.
* Click on the **Create Team** button.
* Fill in the team details (e.g., name, email) and submit the form.
* The newly created team will be associated with the selected organization.

#### Folder and Dashboard Management



Super Admins can provision dashboards for teams by creating folders, assigning teams to those folders, and adding specific dashboards to the folders.

**Creating Folders**



* Navigate to the **Folder Management** page.
* Click on the **Create Folder** button.
* Provide a title for the folder and submit the form.

**Assigning Teams to Folders**



* Select a folder from the **Folder Management** page.
* Use the **Manage Teams** option to assign teams to the folder.
* Choose the team and set the appropriate permissions (View, Edit, or Admin).

**Adding Dashboards to Folders**



* Select a folder from the **Folder Management** page.
* Use the **Manage Dashboards** option to add dashboards to the folder.
* Select dashboards from the list and assign them to the folder.

#### Key Features



* **Team Mapping**: Teams are mapped to organizations for better hierarchy and management.
* **Folder Permissions**: Teams can be assigned specific permissions (View, Edit, Admin) for folders.
* **Dashboard Organization**: Dashboards can be grouped into folders for easy access and management.

This functionality ensures that dashboards are securely and efficiently provisioned to the right teams, enabling seamless collaboration and access control.
