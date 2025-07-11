# DevTide Frontend - Claude.md

## Project Overview
The DevTide Frontend is a modern web UI for the DevTide platform, built with go-app v10 and compiled to WebAssembly. It provides a cyberpunk-themed interface with power-user terminal capabilities for managing cloud infrastructure through GitOps workflows.

## Architecture Overview

### Technology Stack
- **Framework**: go-app v10 (WebAssembly)
- **Server**: Echo web framework
- **Styling**: Tailwind CSS with cyberpunk theme
- **Communication**: gRPC-Web to DevTide Operator
- **State Management**: go-app's built-in state system
- **Terminal**: Integrated xterm.js for power users

### Key Features
1. **Composition Management**: Visual interface for creating and managing infrastructure compositions
2. **Deployment Dashboard**: Real-time status of all deployments across clusters
3. **GitOps Workflow**: Visual representation of Git commits and FluxCD synchronization
4. **Terminal Interface**: Power-user terminal for direct interaction
5. **Multi-tenant Views**: Organization and project-based access control

## Frontend Architecture Design

Based on FRONTEND_ARCHITECTURE.md:

### Component Structure
```
┌─────────────────────────────────────────────────────────┐
│                    App Shell                             │
│  ┌─────────────┬──────────────────┬──────────────────┐ │
│  │   Header    │   Main Content   │     Sidebar      │ │
│  │             │                  │                  │ │
│  │  Logo       │  ┌────────────┐ │   Navigation    │ │
│  │  User Menu  │  │            │ │   Quick Actions │ │
│  │             │  │   Router   │ │                  │ │
│  └─────────────┤  │   Views    │ ├──────────────────┘ │
│                │  │            │ │                    │
│  ┌─────────────┤  └────────────┘ ├──────────────────┐ │
│  │             │                  │                  │ │
│  │  Terminal   │   Status Bar    │   Notifications  │ │
│  │  (xterm.js) │                  │                  │ │
│  └─────────────┴──────────────────┴──────────────────┘ │
└─────────────────────────────────────────────────────────┘
```

### Core Components

#### Dashboard View
- Real-time deployment status grid
- Cluster health indicators
- Resource utilization charts
- Quick action buttons

#### Composition Editor
- YAML editor with syntax highlighting
- Visual parameter builder
- Module dependency graph
- Preview and validation

#### Deployment Manager
- Deployment list with filtering
- Status timeline visualization
- Log viewer with streaming
- Rollback capabilities

#### Terminal Interface
- Full xterm.js integration
- Custom commands for DevTide operations
- Autocompletion and history
- Multi-tab support

## Repository Structure
```
devtide-frontend/
├── cmd/server/           # Echo server entry point
├── pkg/
│   ├── frontend/        # go-app v10 components
│   │   ├── app/        # Main app component
│   │   ├── components/ # Reusable UI components
│   │   ├── views/      # Page-level components
│   │   ├── services/   # gRPC-Web clients
│   │   └── state/      # State management
│   ├── server/         # Echo server setup
│   └── api/            # API client interfaces
├── web/
│   ├── assets/         # Static assets
│   │   ├── styles/    # Tailwind CSS
│   │   ├── fonts/     # Custom fonts
│   │   └── images/    # Icons and graphics
│   └── wasm/          # Compiled WASM output
├── tailwind/           # Tailwind configuration
│   ├── config.js      # Tailwind config
│   └── cyberpunk.js   # Custom theme
└── docs/              # Documentation
```

## Development Workflow

### Prerequisites
- Go 1.20+
- Node.js 18+ (for Tailwind CSS)
- Make
- Docker (optional)

### Local Development
```bash
# Clone the repository
git clone https://github.com/axyzlabs/devtide-frontend.git
cd devtide-frontend

# Install dependencies
go mod download
npm install

# Start development server with hot reload
make dev

# Build Tailwind CSS (in another terminal)
make watch-css
```

### Building
```bash
# Build WASM binary
make build-wasm

# Build server binary
make build-server

# Build Docker image
make docker-build

# Build everything
make build-all
```

### Testing
```bash
# Run Go tests
make test

# Run component tests
make test-components

# Run e2e tests (requires running backend)
make test-e2e
```

## UI Components Gallery

### Design System
Based on UI_COMPONENTS_GALLERY.md:

#### Color Palette
```css
/* Cyberpunk Theme */
--primary: #00ff88;      /* Neon green */
--secondary: #ff006e;    /* Hot pink */
--accent: #00d4ff;       /* Cyan */
--background: #0a0a0a;   /* Deep black */
--surface: #1a1a1a;      /* Dark gray */
--text: #e0e0e0;         /* Light gray */
```

#### Component Examples

##### Status Indicator
```go
type StatusIndicator struct {
    app.Compo
    Status string // "healthy", "warning", "error"
}

func (s *StatusIndicator) Render() app.UI {
    return app.Div().
        Class("status-indicator", s.Status).
        Text(s.Status)
}
```

##### Terminal Component
```go
type Terminal struct {
    app.Compo
    term *xterm.Terminal
}

func (t *Terminal) OnMount(ctx app.Context) {
    t.term = xterm.New(xterm.Options{
        Theme: cyberpunkTheme,
        CursorBlink: true,
    })
}
```

## Communication with Backend

### gRPC-Web Setup
```go
// Connect to operator
conn, err := grpc.DialContext(ctx, operatorEndpoint,
    grpc.WithTransportCredentials(creds),
    grpc.WithDefaultCallOptions(
        grpc.UseCompressor(gzip.Name),
    ),
)

// Create service clients
compositionClient := pb.NewCompositionServiceClient(conn)
deploymentClient := pb.NewDeploymentServiceClient(conn)
```

### Real-time Updates
- WebSocket connection for live status
- Server-sent events for notifications
- Automatic reconnection logic
- Optimistic UI updates

## State Management

### Application State
```go
type AppState struct {
    User         *User
    Organization *Organization
    Deployments  []Deployment
    Compositions []Composition
    Clusters     []Cluster
}

// State updates trigger UI re-renders
func (a *App) UpdateDeployments(deployments []Deployment) {
    a.state.Deployments = deployments
    a.Update()
}
```

## Styling Guidelines

### Tailwind Configuration
```javascript
// tailwind.config.js
module.exports = {
  content: ['./pkg/frontend/**/*.go'],
  theme: {
    extend: {
      colors: {
        'cyber-green': '#00ff88',
        'cyber-pink': '#ff006e',
        'cyber-blue': '#00d4ff',
      },
      fontFamily: {
        'mono': ['Fira Code', 'monospace'],
      },
      animation: {
        'glow': 'glow 2s ease-in-out infinite',
      }
    }
  }
}
```

### Component Styling
- Use Tailwind utility classes
- Cyberpunk theme with neon effects
- Dark mode by default
- Responsive design for all screens

## Performance Optimization

### WASM Optimization
- Minimize binary size with build flags
- Use TinyGo where applicable
- Lazy load heavy components
- Implement virtual scrolling

### Asset Optimization
- Compress images and fonts
- Use WebP format
- Implement progressive loading
- Cache static assets

## Security Considerations

### Authentication
- OAuth2/OIDC integration
- JWT token management
- Secure token storage
- Automatic token refresh

### Authorization
- Role-based access control
- Resource-level permissions
- UI element visibility control
- API request validation

## Testing Strategy

### Component Testing
```go
func TestDashboard(t *testing.T) {
    compo := &Dashboard{
        Deployments: mockDeployments,
    }
    
    // Test rendering
    testapp.AssertComponent(t, compo)
    
    // Test interactions
    testapp.Click(compo, ".deployment-card")
}
```

### E2E Testing
- Cypress for browser automation
- Test critical user flows
- Visual regression testing
- Performance benchmarks

## Deployment

### Docker Deployment
```dockerfile
FROM golang:1.20 AS builder
WORKDIR /app
COPY . .
RUN make build-all

FROM alpine:latest
COPY --from=builder /app/bin/server /server
COPY --from=builder /app/web /web
EXPOSE 8080
CMD ["/server"]
```

### Kubernetes Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: devtide-frontend
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: frontend
        image: axyzlabs/devtide-frontend:latest
        ports:
        - containerPort: 8080
        env:
        - name: OPERATOR_ENDPOINT
          value: "devtide-operator:8080"
```

## Contributing Guidelines

### Code Style
- Follow go-app conventions
- Use meaningful component names
- Keep components small and focused
- Document complex logic

### UI/UX Guidelines
- Maintain cyberpunk aesthetic
- Ensure accessibility (WCAG 2.1)
- Test on multiple browsers
- Mobile-responsive design

### Pull Request Process
1. Create feature branch
2. Implement with tests
3. Update documentation
4. Submit PR with screenshots
5. Pass UI/UX review

## Related Repositories
- [devtide](https://github.com/axyzlabs/devtide): Main operator
- [devtide-agent](https://github.com/axyzlabs/devtide-agent): Cluster agent

## License
[License details to be determined]

---

*Experience the future of infrastructure management with style.*
