Rex VMS

Rex VMS is a web-based visitor management interface developed for Rex Insurance to support visitor registration, appointment handling, access workflows, and front-desk operations.

Overview

Rex VMS provides role-specific interfaces for managing the visitor lifecycle within a corporate environment.

The frontend is organized around three operational portals:

- Security
- Front Desk
- Admin

Each portal provides functionality tailored to its operational responsibilities.

Portals

Security Portal

The Security portal is designed around visitor registration and initial visitor handling.

Its workflow supports activities such as:

- Visitor registration
- Visitor information
- Visitor search
- Identification details
- Initial visitor processing

Front Desk Portal

The Front Desk portal manages appointment and visitor-status workflows.

The interface is designed to allow front-desk staff to:

- Confirm appointments
- Notify hosts
- Track visitor status
- Manage visitor progression
- Complete visitor checkout

The interface explicitly describes the Front Desk workflow as confirming appointments, notifying hosts, and tracking visitors through checkout.

Admin Portal

The Admin portal provides administrative controls for managing the visitor-management system and its operational configuration.

Visitor Workflow

The system is designed around the visitor lifecycle:

Visitor Registration
        │
        ▼
Appointment / Visitor Review
        │
        ▼
Host Notification
        │
        ▼
Visitor Approval
        │
        ▼
Visitor With Host
        │
        ▼
Checkout

Technology Stack

- HTML5
- CSS3
- Vanilla JavaScript
- Responsive Web Design
- REST API integration
- Vercel
- Git & GitHub

Architecture

The frontend is maintained separately from the Flask backend.

┌──────────────────────┐
│      Rex VMS UI      │
├──────────────────────┤
│ Security Portal      │
│ Front Desk Portal    │
│ Admin Portal         │
└──────────┬───────────┘
           │
           │ HTTP / API
           ▼
┌──────────────────────┐
│    Rex VMS Backend   │
│        Flask         │
└──────────┬───────────┘
           │
           ▼
      PostgreSQL

Project Structure

rex-vms/
├── index.html
├── security-portal.html
├── frontdesk-portal.html
├── admin-portal.html
└── README.md

The repository separates the entry interface from the three role-specific portal pages.

Live Demo

Live Application:
https://rex-vms.vercel.app/

Project Context

Rex VMS was developed as an SIWES project during placement at Rex Insurance.

The project was built around the visitor-management workflow used within the organization and translated that operational process into a web-based system.

Project Status

Frontend MVP / Active Project

The frontend serves as the user-facing component of the Rex VMS system.

Related Repository

The server-side implementation is maintained separately in "rex-vms-backend".