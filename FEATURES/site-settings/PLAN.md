# IMPLEMENTATION PLAN - Site Settings Management

## Overview
This plan outlines the step-by-step implementation of the Site Settings Management feature. Each step includes specific actions, files to create/modify, and verification steps.

## Prerequisites
- [ ] Ensure Docker environment is running
- [ ] Verify database connection
- [ ] Confirm all systems are building successfully
- [ ] Have encryption key ready for encrypted_text settings

## Implementation Steps

### STEP 1: Create Database Schema
**Goal**: Define the settings tables structure

**Actions**:
- [ ] Create settings schema file
- [ ] Define setting_groups table
- [ ] Define settings table
- [ ] Define setting_history table
- [ ] Add indexes and triggers

**Files to create**:
- `infrastructure/schemas/db/10_settings.sql`

**Verification**:
```bash
# Review the schema file
cat infrastructure/schemas/db/10_settings.sql
```

---

### STEP 2: Create Database Migrations
**Goal**: Create migrations for schema and initial data

**Actions**:
- [ ] Create schema migration
- [ ] Create data migration with setting groups
- [ ] Add initial System settings

**Files to create**:
- `infrastructure/migrations/db/[timestamp]_create_settings_tables.ts`
- `infrastructure/migrations/db/[timestamp]_seed_settings_data.ts`

**Verification**:
```bash
cd infrastructure
npm run migrate
# Check tables exist in database
docker exec -it infrastructure-postgres-1 psql -U postgres -d museo -c "\dt setting*"
```

---

### STEP 3: Create Domain Entities
**Goal**: Define core business entities

**Actions**:
- [ ] Create SettingGroup entity
- [ ] Create Setting entity with type definitions
- [ ] Create SettingHistory entity
- [ ] Define validation rules interface

**Files to create**:
- `api/src/domain/entities/SettingGroup.ts`
- `api/src/domain/entities/Setting.ts`
- `api/src/domain/entities/SettingHistory.ts`

**Verification**:
```bash
cd api
npm run build
npm run type-check
```

---

### STEP 4: Create Repository Interface
**Goal**: Define data access interface

**Actions**:
- [ ] Create ISettingsRepository interface
- [ ] Define methods for groups, settings, and history
- [ ] Add type definitions for updates and audit

**Files to create**:
- `api/src/domain/repositories/ISettingsRepository.ts`

**Verification**:
```bash
cd api
npm run build
npm run type-check
```

---

### STEP 5: Implement Repository
**Goal**: Create PostgreSQL implementation

**Actions**:
- [ ] Implement all repository methods
- [ ] Handle encryption/decryption for encrypted_text
- [ ] Add transaction support for bulk updates
- [ ] Implement audit logging

**Files to create**:
- `api/src/infrastructure/repositories/PostgresSettingsRepository.ts`

**Verification**:
```bash
cd api
npm run build
npm run test
```

---

### STEP 6: Create DTOs
**Goal**: Define data transfer objects

**Actions**:
- [ ] Create request DTOs for settings operations
- [ ] Create response DTOs
- [ ] Add export/import DTOs

**Files to create**:
- `api/src/application/dto/requests/SettingRequests.ts`
- `api/src/application/dto/responses/SettingResponses.ts`

**Verification**:
```bash
cd api
npm run build
npm run type-check
```

---

### STEP 7: Create Settings Service
**Goal**: Implement core settings service

**Actions**:
- [ ] Create SettingsService with caching
- [ ] Implement getSettingsByGroup method
- [ ] Implement getSettingByKey method
- [ ] Add cache invalidation logic

**Files to create**:
- `api/src/application/services/SettingsService.ts`

**Verification**:
```bash
cd api
npm run build
npm run test
```

---

### STEP 8: Create Validation Service
**Goal**: Implement setting validation

**Actions**:
- [ ] Create validation service
- [ ] Implement type-specific validators
- [ ] Add custom validation rules support

**Files to create**:
- `api/src/infrastructure/services/SettingValidationService.ts`

**Verification**:
```bash
cd api
npm run build
npm run test
```

---

### STEP 9: Create Use Cases - Part 1 (Read Operations)
**Goal**: Implement read use cases

**Actions**:
- [ ] Create GetSettingsByGroupUseCase
- [ ] Create GetSettingByKeyUseCase
- [ ] Create GetSettingHistoryUseCase

**Files to create**:
- `api/src/application/usecases/settings/GetSettingsByGroupUseCase.ts`
- `api/src/application/usecases/settings/GetSettingByKeyUseCase.ts`
- `api/src/application/usecases/settings/GetSettingHistoryUseCase.ts`

**Verification**:
```bash
cd api
npm run build
npm run test
```

---

### STEP 10: Create Use Cases - Part 2 (Write Operations)
**Goal**: Implement write use cases

**Actions**:
- [ ] Create UpdateSettingUseCase
- [ ] Create BulkUpdateSettingsUseCase
- [ ] Create ValidateSettingUseCase

**Files to create**:
- `api/src/application/usecases/settings/UpdateSettingUseCase.ts`
- `api/src/application/usecases/settings/BulkUpdateSettingsUseCase.ts`
- `api/src/application/usecases/settings/ValidateSettingUseCase.ts`

**Verification**:
```bash
cd api
npm run build
npm run test
```

---

### STEP 11: Create Export/Import Services
**Goal**: Implement export/import functionality

**Actions**:
- [ ] Create export service for JSON/XML
- [ ] Create import service with validation
- [ ] Add format converters

**Files to create**:
- `api/src/infrastructure/services/SettingExportService.ts`
- `api/src/infrastructure/services/SettingImportService.ts`

**Verification**:
```bash
cd api
npm install xml2js js2xmlparser
npm run build
```

---

### STEP 12: Create Export/Import Use Cases
**Goal**: Implement export/import use cases

**Actions**:
- [ ] Create ExportSettingsUseCase
- [ ] Create ImportSettingsUseCase
- [ ] Add validation and error handling

**Files to create**:
- `api/src/application/usecases/settings/ExportSettingsUseCase.ts`
- `api/src/application/usecases/settings/ImportSettingsUseCase.ts`

**Verification**:
```bash
cd api
npm run build
npm run test
```

---

### STEP 13: Create Admin Settings Controller
**Goal**: Implement API endpoints

**Actions**:
- [ ] Create controller with all endpoints
- [ ] Add error handling
- [ ] Implement file upload for import

**Files to create**:
- `api/src/api/controllers/AdminSettingsController.ts`

**Verification**:
```bash
cd api
npm run build
npm run lint
```

---

### STEP 14: Create Validation Schemas
**Goal**: Add request validation

**Actions**:
- [ ] Create schemas for all endpoints
- [ ] Add type-specific validation
- [ ] Include file upload validation

**Files to create**:
- `api/src/api/schemas/settingSchemas.ts`

**Verification**:
```bash
cd api
npm run build
```

---

### STEP 15: Create Admin Settings Routes
**Goal**: Define API routes

**Actions**:
- [ ] Create route definitions
- [ ] Add validation middleware
- [ ] Configure file upload middleware

**Files to create**:
- `api/src/api/routes/adminSettingsRoutes.ts`

**Files to modify**:
- `api/src/api/routes/index.ts` (add settings routes)

**Verification**:
```bash
cd api
npm run build
npm run dev
# Test with curl or Postman
curl http://localhost:8000/api/v1/admin/settings/groups
```

---

### STEP 16: Register Dependencies in Container
**Goal**: Wire up all services and use cases

**Actions**:
- [ ] Register SettingsRepository
- [ ] Register SettingsService
- [ ] Register validation and export/import services
- [ ] Register all use cases
- [ ] Register controller

**Files to modify**:
- `api/src/infrastructure/config/container.ts`

**Verification**:
```bash
cd api
npm run build
npm start
```

---

### STEP 17: Create Web Admin Types
**Goal**: Define TypeScript types for frontend

**Actions**:
- [ ] Create setting types
- [ ] Add API response types
- [ ] Define validation types

**Files to create**:
- `web-admin/src/shared/types/settings.ts`

**Files to modify**:
- `web-admin/src/shared/types/api.ts`

**Verification**:
```bash
cd web-admin
npm run type-check
```

---

### STEP 18: Create Setting Input Components
**Goal**: Build dynamic input components

**Actions**:
- [ ] Create SettingInput atom for all types
- [ ] Create SettingLabel atom
- [ ] Create SettingValidationMessage atom

**Files to create**:
- `web-admin/src/shared/components/atoms/SettingInput/index.tsx`
- `web-admin/src/shared/components/atoms/SettingLabel/index.tsx`
- `web-admin/src/shared/components/atoms/SettingValidationMessage/index.tsx`

**Verification**:
```bash
cd web-admin
npm run build
npm run lint
```

---

### STEP 19: Create Setting Field Components
**Goal**: Build field-level components

**Actions**:
- [ ] Create SettingField molecule
- [ ] Create SettingGroupCard molecule
- [ ] Create SettingHistoryItem molecule

**Files to create**:
- `web-admin/src/shared/components/molecules/SettingField/index.tsx`
- `web-admin/src/shared/components/molecules/SettingGroupCard/index.tsx`
- `web-admin/src/shared/components/molecules/SettingHistoryItem/index.tsx`

**Verification**:
```bash
cd web-admin
npm run build
```

---

### STEP 20: Create Settings Form Components
**Goal**: Build form-level components

**Actions**:
- [ ] Create SettingsForm organism
- [ ] Create SettingsImportDialog organism
- [ ] Create SettingsExportDialog organism
- [ ] Create SettingHistoryModal organism

**Files to create**:
- `web-admin/src/shared/components/organisms/SettingsForm/index.tsx`
- `web-admin/src/shared/components/organisms/SettingsImportDialog/index.tsx`
- `web-admin/src/shared/components/organisms/SettingsExportDialog/index.tsx`
- `web-admin/src/shared/components/organisms/SettingHistoryModal/index.tsx`

**Verification**:
```bash
cd web-admin
npm run build
```

---

### STEP 21: Create Settings Service
**Goal**: Implement API client

**Actions**:
- [ ] Create AdminSettingsService
- [ ] Implement all API methods
- [ ] Add error handling

**Files to create**:
- `web-admin/src/shared/services/AdminSettingsService.ts`

**Verification**:
```bash
cd web-admin
npm run build
npm run type-check
```

---

### STEP 22: Update Admin Store
**Goal**: Add settings state management

**Actions**:
- [ ] Add settings state
- [ ] Add actions for CRUD operations
- [ ] Add import/export actions

**Files to modify**:
- `web-admin/src/shared/store/AdminStore.ts`

**Verification**:
```bash
cd web-admin
npm run type-check
```

---

### STEP 23: Create Settings Pages
**Goal**: Build settings UI pages

**Actions**:
- [ ] Create main settings page
- [ ] Create settings group page
- [ ] Add navigation

**Files to create**:
- `web-admin/src/app/settings/page.tsx`
- `web-admin/src/app/settings/[groupId]/page.tsx`

**Verification**:
```bash
cd web-admin
npm run dev
# Navigate to http://localhost:3001/settings
```

---

### STEP 24: Add Styles
**Goal**: Style settings components

**Actions**:
- [ ] Create settings-specific styles
- [ ] Add responsive design
- [ ] Ensure consistent with design system

**Files to create**:
- `web-admin/src/styles/components/_settings.scss`
- `web-admin/src/styles/pages/_settings.scss`

**Files to modify**:
- `web-admin/src/styles/main.scss` (import new styles)

**Verification**:
```bash
cd web-admin
npm run build
# Visual inspection in browser
```

---

### STEP 25: Add Translations
**Goal**: Add UI text translations

**Actions**:
- [ ] Add settings-related translations
- [ ] Add validation messages
- [ ] Add success/error messages

**Files to modify**:
- `web-admin/src/i18n/locales/en.json`
- `web-admin/src/i18n/locales/es.json`

**Verification**:
```bash
cd web-admin
npm run build
# Test language switching
```

---

### STEP 26: Integration Testing
**Goal**: Test complete feature end-to-end

**Actions**:
- [ ] Test all setting types
- [ ] Test validation rules
- [ ] Test export/import functionality
- [ ] Test audit history
- [ ] Test caching behavior
- [ ] Test error scenarios

**Verification**:
```bash
# Start all services
cd infrastructure
sudo docker compose up

# Run API tests
cd ../api
npm test

# Manual testing in browser
# 1. Navigate to settings
# 2. Edit various setting types
# 3. Export settings
# 4. Import settings
# 5. View history
```

---

## Post-Implementation Checklist

- [ ] All tests passing
- [ ] No TypeScript errors
- [ ] No ESLint warnings
- [ ] Settings properly cached
- [ ] Encryption working for encrypted_text
- [ ] Audit logs recording changes
- [ ] Export/import working for JSON and XML
- [ ] UI responsive on all devices
- [ ] Translations complete
- [ ] Performance acceptable

## Rollback Plan

If issues arise during deployment:

1. **Database**: Migration includes down() method for rollback
2. **API**: Previous version can be redeployed
3. **Frontend**: Feature can be hidden via navigation
4. **Cache**: Redis cache can be flushed

## Success Criteria

- [ ] Settings load in < 500ms
- [ ] Updates apply immediately
- [ ] Export/import handles 100+ settings
- [ ] Validation prevents invalid data
- [ ] Audit trail captures all changes
- [ ] No regression in existing functionality

## Notes

- Each step should be completed and verified before moving to the next
- Run build, type-check, and lint after each major change
- Commit changes after each successful step
- Test incrementally to catch issues early
- Consider feature flag for gradual rollout