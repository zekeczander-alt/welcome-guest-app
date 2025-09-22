# Prompt Library Document

**Document Version:** 1.0  
**Date Created:** September 15, 2025  
**Author(s):** AI-Generated  
**Project Name:** Guest Link - A WhatsApp-based Hospitality App

---

## 1. Introduction

**Purpose:**  
This Prompt Library Document serves as a curated collection of AI prompts for Guest Link, a WhatsApp-integrated hospitality management platform. It organizes prompts into categories aligned with our core features: welcome page generation, WhatsApp integration, and service management. The library supports Vibe Coding by providing reusable prompts that ensure consistency with our technical requirements (Next.js 13+, TypeScript, Tailwind CSS) and compliance standards (WCAG 2.1 AA, GDPR). The prompts evolve based on development feedback and changing project needs.

**Scope:**  
Covers prompts for generating frontend components, backend APIs, WhatsApp integration code, test suites, and documentation. Includes specific prompts for welcome page generation and service management features. Excludes general AI usage guidelines not specific to hospitality management features.

**Assumptions:**  
- AI tools support structured prompts with clear outputs
- Prompts align with our tech stack (Next.js 13+, TypeScript, Tailwind, PostgreSQL)
- WhatsApp Business API remains stable and accessible
- Developers validate AI outputs against project specifications
- Generated code follows accessibility and data privacy standards

---

## 2. Prompt Organization and Usage

- **Organization:** Prompts are grouped by feature area and development phase:
  - Welcome Page Generation
  - WhatsApp Integration
  - Service Management
  - Testing and Validation
  - Documentation Generation

- **Variables:** Use consistent placeholders:
  - `{property_id}`: Unique property identifier
  - `{stay_id}`: Unique stay/booking identifier
  - `{service_type}`: Type of service (e.g., cleaning, dining)
  - `{template_name}`: WhatsApp message template name
  - `{component_name}`: React component name

- **Usage Guidelines:**
  - Start with feature-specific prompts from relevant categories
  - Replace variables with actual values from context
  - Validate outputs against Style Guide and API Specification
  - Add successful prompts back to library with examples
  - Include accessibility and internationalization requirements

- **Storage:**
  - Maintain in `prompts/` directory with clear categorization
  - Version control through Git
  - Include successful outputs as examples
  - Document any required context or dependencies

---

## 3. Prompt Categories

### 3.1 Welcome Page Generation Prompts

- **Prompt 1: Generate Welcome Page Component**
  - **Description:** Creates a dynamic welcome page component with property information
  - **Variables:**
    - `{property_info}`: Property details (name, location, amenities)
    - `{branding_config}`: Property branding settings
    - `{languages}`: Supported languages
  - **Template:**
    ```
    Generate a Next.js 13+ welcome page component for {property_info} using TypeScript and Tailwind CSS. Include branding from {branding_config}. Support languages: {languages}. Ensure WCAG 2.1 AA compliance and responsive design. Output as app/welcome/[stayId]/page.tsx.
    ```
  - **Sample Output (condensed):**
    ```tsx
    import { getBranding } from '@/lib/branding';
    import { WelcomeHeader, ServiceList } from '@/components';

    export default async function WelcomePage({ params }: { params: { stayId: string } }) {
      const branding = await getBranding(params.stayId);
      
      return (
        <main className="min-h-screen bg-gradient-to-b from-brand-100 to-brand-50">
          <WelcomeHeader branding={branding} />
          <ServiceList stayId={params.stayId} />
          <WhatsAppConnect stayId={params.stayId} />
        </main>
      );
    }
    ```

- **Prompt 2: Generate Service Card Component**
  - **Description:** Creates a service display card for welcome pages
  - **Variables:**
    - `{service_type}`: Type of service to display
    - `{style_theme}`: Property's style theme
  - **Template:**
    ```
    Create a React component for displaying a {service_type} service card using Tailwind CSS and TypeScript. Follow {style_theme} from style guide. Include booking functionality and WhatsApp integration. Output as components/services/ServiceCard.tsx.
    ```

### 3.2 WhatsApp Integration Prompts

- **Prompt 3: Generate WhatsApp Webhook Handler**
  - **Description:** Creates API route for WhatsApp webhook events
  - **Variables:**
    - `{event_type}`: Type of webhook event
    - `{message_handler}`: Message processing function
  - **Template:**
    ```
    Generate a Next.js API route for handling WhatsApp {event_type} webhooks. Use TypeScript and implement {message_handler} for processing. Include security validation and error handling. Output as app/api/whatsapp/webhook/route.ts.
    ```

- **Prompt 4: Generate Message Template Manager**
  - **Description:** Creates utility for managing WhatsApp message templates
  - **Variables:**
    - `{template_type}`: Type of message template
    - `{variables}`: Template variables
  - **Template:**
    ```
    Create a TypeScript utility for managing {template_type} WhatsApp templates with variables: {variables}. Include validation and localization support. Output as lib/whatsapp/templates.ts.
    ```

### 3.3 Service Management Prompts

- **Prompt 5: Generate Service Request Handler**
  - **Description:** Creates API route for service requests
  - **Variables:**
    - `{service_type}`: Type of service
    - `{validation_rules}`: Input validation rules
  - **Template:**
    ```
    Generate a Next.js API route for handling {service_type} service requests. Include {validation_rules} using Zod. Implement PostgreSQL storage and WhatsApp notifications. Output as app/api/services/route.ts.
    ```

- **Prompt 6: Generate Service Dashboard Component**
  - **Description:** Creates staff dashboard for service management
  - **Variables:**
    - `{view_type}`: Dashboard view type (list/calendar)
    - `{filters}`: Service filters to include
  - **Template:**
    ```
    Create a React component for the {view_type} service dashboard with {filters}. Use TypeScript and Tailwind CSS. Include real-time updates via WebSocket. Output as components/dashboard/ServiceDashboard.tsx.
    ```

### 3.4 Testing Prompts

- **Prompt 7: Generate Component Tests**
  - **Description:** Creates test suite for React components
  - **Variables:**
    - `{component_name}`: Component to test
    - `{test_scenarios}`: Test cases to cover
  - **Template:**
    ```
    Generate Jest tests for {component_name} covering {test_scenarios}. Use React Testing Library. Include accessibility tests and edge cases. Output as __tests__/components/{component_name}.test.tsx.
    ```

- **Prompt 8: Generate API Integration Tests**
  - **Description:** Creates integration tests for API endpoints
  - **Variables:**
    - `{endpoint_path}`: API endpoint to test
    - `{test_cases}`: Test scenarios to cover
  - **Template:**
    ```
    Create integration tests for {endpoint_path} covering {test_cases}. Use Supertest. Include authentication, error cases, and rate limiting tests. Output as __tests__/api/{endpoint_path}.test.ts.
    ```

### 3.5 Documentation Prompts

- **Prompt 9: Generate API Documentation**
  - **Description:** Creates OpenAPI documentation for endpoints
  - **Variables:**
    - `{endpoint_group}`: Group of endpoints
    - `{auth_requirements}`: Authentication requirements
  - **Template:**
    ```
    Generate OpenAPI documentation for {endpoint_group} endpoints. Include {auth_requirements}, request/response schemas, and examples. Output as docs/api/{endpoint_group}.md.
    ```

- **Prompt 10: Generate Component Documentation**
  - **Description:** Creates component documentation with examples
  - **Variables:**
    - `{component_name}`: Component to document
    - `{props_interface}`: Component props interface
  - **Template:**
    ```
    Create documentation for {component_name} with {props_interface}. Include usage examples, props table, and accessibility notes. Output as docs/components/{component_name}.md.
    ```

---

## 4. Prompt Best Practices

- **Clarity:**
  - Specify exact tech stack requirements (Next.js 13+, TypeScript, Tailwind)
  - Include accessibility requirements (WCAG 2.1 AA)
  - Reference relevant sections of project documentation

- **Context:**
  - Link to Style Guide for consistent theming
  - Reference API Specification for endpoint patterns
  - Include data privacy requirements for guest information

- **Validation:**
  - Test outputs against Component Breakdown specs
  - Verify WhatsApp integration requirements
  - Check accessibility compliance
  - Validate data handling for GDPR compliance

- **Example Workflow:**
  1. Use welcome page prompts to generate base structure
  2. Add WhatsApp integration components
  3. Implement service management features
  4. Generate tests for new components
  5. Create documentation for added features

---

## 5. Risks and Assumptions

**Assumptions:**
- AI tools understand hospitality industry context
- WhatsApp Business API patterns are consistent
- Generated code follows project architecture
- Developers have access to all required APIs
- Test environments support WhatsApp integration

**Risks:**
- WhatsApp API changes affecting integration code
  - Mitigation: Version check in prompts
- Inconsistent component styling
  - Mitigation: Strict Style Guide references
- Privacy compliance gaps
  - Mitigation: Include GDPR checks
- Performance issues in generated code
  - Mitigation: Include optimization guidelines

---

## 6. Revision History
- September 15, 2025 - v1.0: Initial prompt library document
- Future updates will be logged here

---

**Document Purpose Recap:** This Prompt Library Document provides a structured collection of AI prompts specifically tailored for Guest Link's development needs. It ensures consistency, maintainability, and compliance while accelerating development through Vibe Coding.