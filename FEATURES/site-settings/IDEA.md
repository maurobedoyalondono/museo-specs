I want to implement a brand new feature, there is nothing like it in the site, but you need to follow the architecture of    │
│   each:\                                                                                                                       │
│   database:\                                                                                                                   │
│   - Define schemas and sample data\ '/home/negrito/src/projects/Museo/infrastructure/schemas/db'                               │
│   - Create migrations: one for schemas, one for sample data\                                                                   │
│   '/home/negrito/src/projects/Museo/infrastructure/migrations/db'                                                              │
│   \                                                                                                                            │
│   web-admin\\                                                                                                                  │
│   - check '/home/negrito/src/projects/Museo/web-admin'                                                                         │
│   \                                                                                                                            │
│   api: clean architecture, check others admin routes                                                                           │
│   '/home/negrito/src/projects/Museo/api/src/api/routes/adminCategoryRoutes.ts'\                                                │
│   \                                                                                                                            │
│   Concept Settings.\                                                                                                           │
│   Setting are always grouped, just one level groups: System,Payment, Security, etc (just samples)\                             │
│   For each group we can add as many settings as necessary.\                                                                    │
│   Each setting can have type: text, url,encrypted_text,email, domain.  Help me find more, what might be typical for            │
│   ecommerce-\                                                                                                                  │
│   \                                                                                                                            │
│   It is important to leave a clear service like SettingsService(getElementsByGroup) so any other element in the api system     │
│   can use it.\                                                                                                                 │
│   \                                                                                                                            │
│   Goal: Database, api, UI (CRUD) from web-admin\                                                                               │
│   \                                                                                                                            │
│   You need to follow specs plan.\ '/home/negrito/src/projects/Museo/specs/CLAUDE.md'\                                          │
│   \                                                                                                                            │
│   Sample feature implemented this way: '/home/negrito/src/projects/Museo/specs/FEATURES/admin/master-products'  