
┌──────────────────────────────────────────────────────────────────────────────┐
│                        EXTERNAL USER HITS amazon.com                        │
└──────────────────────────────────────────────────────────────────────────────┘

  STEP 1: DNS Resolution
  ──────────────────────
  User Browser ──► amazon.com
                       │
                       ▼
                   Route 53 (DNS)
                       │
                   Resolves to NLB's static IP
                   (e.g., 52.10.20.30)
                       │
                       ▼

  STEP 2: TCP Connection to NLB (Layer 4)
  ────────────────────────────────────────
  ┌─────────────────────────────────┐
  │   NETWORK LOAD BALANCER (NLB)   │
  │   ─────────────────────────────  │
  │   • Static IP: 52.10.20.30      │
  │   • Listener: TCP 443           │
  │   • Internet-facing             │
  │   • Does NOT inspect HTTP       │
  │   • Ultra-low latency           │
  │   • Preserves client source IP  │
  │                                  │
  │   Action: Forwards raw TCP       │
  │   packets to ALB target group    │
  └──────────────┬──────────────────┘
                 │
                 │  TCP passthrough (no TLS termination here)
                 ▼

  STEP 3: ALB Receives Traffic (Layer 7)
  ────────────────────────────────────────
  ┌──────────────────────────────────────────────────────────┐
  │          APPLICATION LOAD BALANCER (ALB)                  │
  │          ──────────────────────────────                   │
  │   • Internal ALB (private subnets)                        │
  │   • TLS Termination (decrypts HTTPS using ACM cert)       │
  │   • Inspects HTTP headers, path, host                     │
  │   • Applies WAF rules (block SQL injection, XSS, etc.)   │
  │                                                           │
  │   Routing Decision:                                       │
  │   ┌─────────────────────────────────────────────────┐     │
  │   │  Host: amazon.com    Path: /           → App A  │     │
  │   │  Host: amazon.com    Path: /api/*      → App B  │     │
  │   │  Host: amazon.com    Path: /checkout/* → App C  │     │
  │   │  Host: m.amazon.com  Path: /*          → App D  │     │
  │   └─────────────────────────────────────────────────┘     │
  └──────────────────────┬────────────────────────────────────┘
                         │
                         │  HTTP request routed to correct Target Group
                         ▼

  STEP 4: Target Group → EKS Pods
  ────────────────────────────────
  ┌──────────────────────────────────────────────────────────┐
  │                    EKS CLUSTER                            │
  │                                                           │
  │   Target Group A (Pod IPs)     Target Group B (Pod IPs)   │
  │   ┌─────────┐ ┌─────────┐     ┌─────────┐ ┌─────────┐   │
  │   │ Pod A-1 │ │ Pod A-2 │     │ Pod B-1 │ │ Pod B-2 │   │
  │   │ Frontend│ │ Frontend│     │  API    │ │  API    │   │
  │   │10.0.1.5 │ │10.0.2.8 │     │10.0.1.9 │ │10.0.2.3 │   │
  │   └─────────┘ └─────────┘     └─────────┘ └─────────┘   │
  │                                                           │
  │   Target Group C (Pod IPs)     Target Group D (Pod IPs)   │
  │   ┌─────────┐ ┌─────────┐     ┌─────────┐ ┌─────────┐   │
  │   │ Pod C-1 │ │ Pod C-2 │     │ Pod D-1 │ │ Pod D-2 │   │
  │   │Checkout │ │Checkout │     │ Mobile  │ │ Mobile  │   │
  │   │10.0.1.7 │ │10.0.2.6 │     │10.0.1.4 │ │10.0.2.1 │   │
  │   └─────────┘ └─────────┘     └─────────┘ └─────────┘   │
  └──────────────────────────────────────────────────────────┘

  STEP 5: Response travels back
  ──────────────────────────────
  Pod → ALB (encrypts response) → NLB (TCP passthrough) → User Browser

