# Component Breakdown Document

**Document Version:** 1.0  
**Date Created:** September 15, 2025  
**Author(s):** AI-Generated  
**Project Name:** Guest Link - A WhatsApp-based Hospitality App

---

## 1. Introduction
This Component Breakdown Document decomposes the UI and application structure of Guest Link into its constituent elements: pages, layouts, and reusable components. Building upon the Requirements Specification and Architecture Design documents, it provides detailed implementation guidelines for each component, ensuring consistency and modularity in development.

The document focuses on the key features identified in our requirements:
- Welcome page generation and management
- WhatsApp integration and communication
- Service request handling
- Staff dashboard and task management
- Analytics and reporting interfaces

This breakdown supports iterative development through Vibe Coding by enabling modular component generation and testing. The structure assumes Next.js 13+ with App Router, React, and TypeScript, promoting reusability and maintainability.

---

## 2. Hierarchical Structure Overview
The application follows a hierarchical structure organized by feature areas:

```
Pages
  - /welcome/[stayId] (Guest Welcome Page)
  - /dashboard (Host Dashboard)
  - /services (Service Management)
  - /stays (Stay Management)
  - /analytics (Reports and Analytics)
Layouts
  - RootLayout
  - DashboardLayout
  - WelcomeLayout
Components
  - UI Primitives
    - Buttons, Inputs, Cards
  - Feature Components
    - WelcomePageEditor
    - ServiceRequestCard
    - WhatsAppChat
  - Dashboard Components
    - StayList
    - ServiceQueue
    - Analytics Charts
```

---

## 3. Pages

### 3.1 Welcome Pages
- **Page: /welcome/[stayId] (app/welcome/[stayId]/page.tsx)**
  - **Description:** Dynamic welcome page for guests, accessible via unique URL
  - **Responsibilities:**
    - Load stay-specific content and branding
    - Display property information and services
    - Handle service requests
    - Integrate WhatsApp communication
  - **Composed Elements:**
    - WelcomeLayout
    - PropertyInfo
    - ServiceCatalog
    - WhatsAppConnect
  - **Props:**
    ```typescript
    interface WelcomePageProps {
      params: { stayId: string };
    }
    ```
  - **Pseudo-Code:**
    ```typescript
    async function WelcomePage({ params }: WelcomePageProps) {
      const stay = await getStayDetails(params.stayId);
      const services = await getAvailableServices(stay.propertyId);
      
      return (
        <WelcomeLayout brandingData={stay.property.branding}>
          <PropertyInfo stay={stay} />
          <ServiceCatalog services={services} stayId={stay.id} />
          <WhatsAppConnect 
            whatsappNumber={stay.whatsappNumber}
            propertyName={stay.property.name}
          />
        </WelcomeLayout>
      );
    }
    ```

### 3.2 Host Dashboard Pages
- **Page: /dashboard (app/dashboard/page.tsx)**
  - **Description:** Main dashboard for property hosts
  - **Responsibilities:**
    - Display active stays
    - Show pending service requests
    - Present key metrics
    - List recent communications
  - **Composed Elements:**
    - DashboardLayout
    - ActiveStaysList
    - ServiceRequestQueue
    - MetricsDisplay
  - **Pseudo-Code:**
    ```typescript
    async function DashboardPage() {
      const activeStays = await getActiveStays();
      const pendingRequests = await getPendingRequests();
      const metrics = await getDashboardMetrics();
      
      return (
        <DashboardLayout>
          <MetricsDisplay data={metrics} />
          <div className="grid grid-cols-2 gap-4">
            <ActiveStaysList stays={activeStays} />
            <ServiceRequestQueue requests={pendingRequests} />
          </div>
        </DashboardLayout>
      );
    }
    ```

### 3.3 Service Management Pages
- **Page: /services (app/services/page.tsx)**
  - **Description:** Service configuration and management
  - **Responsibilities:**
    - List available services
    - Configure service options
    - Manage staff assignments
    - Track service metrics
  - **Composed Elements:**
    - ServiceList
    - ServiceEditor
    - StaffAssignment
  - **Pseudo-Code:**
    ```typescript
    async function ServicesPage() {
      const services = await getServices();
      const staff = await getStaff();
      
      return (
        <DashboardLayout>
          <ServiceList 
            services={services}
            onServiceUpdate={updateService}
          />
          <ServiceEditor />
          <StaffAssignment staff={staff} />
        </DashboardLayout>
      );
    }
    ```

---

## 4. Layouts

### 4.1 RootLayout (app/layout.tsx)
- **Description:** Base layout for the entire application
- **Responsibilities:**
  - Global providers setup
  - Authentication state
  - Theme management
  - Navigation structure
- **Pseudo-Code:**
  ```typescript
  export default function RootLayout({
    children
  }: {
    children: React.ReactNode
  }) {
    return (
      <html lang="en">
        <body>
          <Providers>
            <Navigation />
            <main>{children}</main>
          </Providers>
        </body>
      </html>
    );
  }
  ```

### 4.2 DashboardLayout (app/dashboard/layout.tsx)
- **Description:** Layout for host-facing pages
- **Responsibilities:**
  - Sidebar navigation
  - Real-time updates
  - Authentication check
- **Pseudo-Code:**
  ```typescript
  async function DashboardLayout({
    children
  }: {
    children: React.ReactNode
  }) {
    const user = await getCurrentUser();
    if (!user) redirect('/login');

    return (
      <div className="grid grid-cols-[280px_1fr]">
        <Sidebar />
        <div className="p-6">{children}</div>
      </div>
    );
  }
  ```

### 4.3 WelcomeLayout (app/welcome/layout.tsx)
- **Description:** Layout for guest welcome pages
- **Responsibilities:**
  - Brand theming
  - Mobile responsiveness
  - WhatsApp integration
- **Pseudo-Code:**
  ```typescript
  function WelcomeLayout({
    children,
    brandingData
  }: {
    children: React.ReactNode,
    brandingData: BrandingConfig
  }) {
    return (
      <div className="max-w-4xl mx-auto p-4" style={brandingData.styles}>
        <header>
          <BrandHeader {...brandingData} />
        </header>
        {children}
        <WhatsAppFloatingButton />
      </div>
    );
  }
  ```

---

## 5. Components

### 5.1 UI Primitives (components/ui/)

- **Component: Button**
  ```typescript
  interface ButtonProps {
    variant: 'primary' | 'secondary' | 'outline';
    children: React.ReactNode;
    onClick?: () => void;
    loading?: boolean;
    disabled?: boolean;
  }

  function Button({
    variant = 'primary',
    children,
    onClick,
    loading,
    disabled
  }: ButtonProps) {
    return (
      <button
        className={getButtonStyles(variant)}
        onClick={onClick}
        disabled={disabled || loading}
      >
        {loading ? <Spinner /> : children}
      </button>
    );
  }
  ```

- **Component: ServiceCard**
  ```typescript
  interface ServiceCardProps {
    service: {
      id: string;
      name: string;
      description: string;
      price?: number;
      availability: boolean;
    };
    onRequest: (serviceId: string) => void;
  }

  function ServiceCard({ service, onRequest }: ServiceCardProps) {
    return (
      <div className="border rounded-lg p-4 shadow-sm">
        <h3>{service.name}</h3>
        <p>{service.description}</p>
        {service.price && <p>Price: ${service.price}</p>}
        <Button
          variant="primary"
          onClick={() => onRequest(service.id)}
          disabled={!service.availability}
        >
          Request Service
        </Button>
      </div>
    );
  }
  ```

### 5.2 Feature Components (components/features/)

- **Component: WelcomePageEditor**
  ```typescript
  interface WelcomePageEditorProps {
    stayId: string;
    initialContent: WelcomePageContent;
    onSave: (content: WelcomePageContent) => Promise<void>;
  }

  function WelcomePageEditor({
    stayId,
    initialContent,
    onSave
  }: WelcomePageEditorProps) {
    const [content, setContent] = useState(initialContent);
    
    return (
      <div className="space-y-4">
        <SectionEditor
          title="Property Information"
          content={content.propertyInfo}
          onChange={(propertyInfo) => setContent({ ...content, propertyInfo })}
        />
        <ServiceSelector
          selectedServices={content.services}
          onChange={(services) => setContent({ ...content, services })}
        />
        <Button onClick={() => onSave(content)}>Save Changes</Button>
      </div>
    );
  }
  ```

- **Component: WhatsAppChat**
  ```typescript
  interface WhatsAppChatProps {
    stayId: string;
    messages: Message[];
    onSendMessage: (message: string) => Promise<void>;
  }

  function WhatsAppChat({
    stayId,
    messages,
    onSendMessage
  }: WhatsAppChatProps) {
    const [newMessage, setNewMessage] = useState('');
    
    return (
      <div className="flex flex-col h-[500px]">
        <div className="flex-1 overflow-y-auto">
          {messages.map((message) => (
            <ChatMessage key={message.id} {...message} />
          ))}
        </div>
        <MessageInput
          value={newMessage}
          onChange={setNewMessage}
          onSend={() => {
            onSendMessage(newMessage);
            setNewMessage('');
          }}
        />
      </div>
    );
  }
  ```

### 5.3 Dashboard Components (components/dashboard/)

- **Component: ServiceRequestQueue**
  ```typescript
  interface ServiceRequestQueueProps {
    requests: ServiceRequest[];
    onStatusUpdate: (requestId: string, status: RequestStatus) => Promise<void>;
  }

  function ServiceRequestQueue({
    requests,
    onStatusUpdate
  }: ServiceRequestQueueProps) {
    const sortedRequests = useMemo(
      () => sortByPriority(requests),
      [requests]
    );
    
    return (
      <div className="space-y-2">
        {sortedRequests.map((request) => (
          <RequestCard
            key={request.id}
            request={request}
            onStatusUpdate={onStatusUpdate}
          />
        ))}
      </div>
    );
  }
  ```

---

## 6. Integration Guidelines

### State Management
- Use React Query for server state
- Zustand for global UI state
- Context for theme and auth
- Local state for form inputs

### Data Flow
- Server Components for data fetching
- Client Components for interactivity
- Props for component communication
- Events for user interactions

### Error Handling
- ErrorBoundary for component errors
- Toast notifications for API errors
- Fallback UI for loading states
- Retry mechanisms for failed requests

### Performance
- Lazy loading for large components
- Memoization for expensive calculations
- Image optimization with next/image
- Incremental Static Regeneration for welcome pages

---

## 7. Risks and Assumptions

### Assumptions
- Next.js 13+ with App Router
- TypeScript for type safety
- Tailwind CSS for styling
- Modern browser support
- Stable WhatsApp API

### Risks
- Complex state management in real-time features
  - Mitigation: Clear data flow patterns
- Performance with many concurrent users
  - Mitigation: Proper caching and optimization
- Mobile responsiveness challenges
  - Mitigation: Mobile-first design approach
- WhatsApp API limitations
  - Mitigation: Fallback communication methods

---

## 8. Revision History
- September 15, 2025 - v1.0 - Initial component breakdown document
- Future updates will be logged here

---

**Document Purpose Recap:** This Component Breakdown Document provides a detailed blueprint for implementing Guest Link's UI components, ensuring consistency and modularity while facilitating efficient development through Vibe Coding.