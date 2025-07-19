# IMPLEMENTATION PLAN - Master Products Management

## Overview
This plan outlines the step-by-step implementation of the Master Products Management feature. Each step includes specific actions, files to create/modify, and verification steps.

## Prerequisites
- [ ] Ensure Docker environment is running
- [ ] Verify database connection
- [ ] Have AWS S3 credentials ready
- [ ] Confirm all systems are building successfully

## Implementation Steps

### STEP 1: Database Migration for Images Field
**Goal**: Add images field to product_masters table

**Actions**:
- [ ] Create new migration file for adding images column
- [ ] Test migration locally
- [ ] Verify rollback works

**Files to create/modify**:
- `infrastructure/migrations/db/[timestamp]_add_product_master_images.ts`

**Verification**:
```bash
cd infrastructure
npm run migrate
npm run migrate:down
npm run migrate
```

---

### STEP 2: Update API Domain Models and DTOs
**Goal**: Update product-related types to include images

**Actions**:
- [ ] Update ProductMaster interface to include images
- [ ] Update DTOs for create/update requests
- [ ] Add image-related types

**Files to modify**:
- `api/src/domain/entities/Product.ts`
- `api/src/application/dtos/product/ProductDTOs.ts`

**Verification**:
```bash
cd api
npm run build
npm run type-check
```

---

### STEP 3: Implement S3 Repository and Service
**Goal**: Create S3 integration for image uploads

**Actions**:
- [ ] Create S3Repository interface in domain layer
- [ ] Implement S3Repository in infrastructure layer
- [ ] Create S3Service for image operations
- [ ] Add AWS SDK dependency

**Files to create**:
- `api/src/domain/repositories/S3Repository.ts`
- `api/src/infrastructure/repositories/S3RepositoryImpl.ts`
- `api/src/infrastructure/services/S3Service.ts`

**Verification**:
```bash
cd api
npm install aws-sdk
npm run build
npm run type-check
```

---

### STEP 4: Create List and Get Use Cases
**Goal**: Implement missing read operations for product masters

**Actions**:
- [ ] Create GetProductMastersUseCase
- [ ] Create GetProductMasterByIdUseCase
- [ ] Update ProductRepository interface
- [ ] Implement repository methods

**Files to create**:
- `api/src/application/usecases/product/GetProductMastersUseCase.ts`
- `api/src/application/usecases/product/GetProductMasterByIdUseCase.ts`

**Files to modify**:
- `api/src/domain/repositories/ProductRepository.ts`
- `api/src/infrastructure/repositories/PostgresProductRepository.ts`

**Verification**:
```bash
cd api
npm run build
npm run test
```

---

### STEP 5: Implement Image Upload Use Case
**Goal**: Create use case for uploading images to S3

**Actions**:
- [ ] Create UploadProductImageUseCase
- [ ] Add image validation logic
- [ ] Integrate with S3Repository

**Files to create**:
- `api/src/application/usecases/product/UploadProductImageUseCase.ts`

**Verification**:
```bash
cd api
npm run build
npm run test
```

---

### STEP 6: Create Export/Import Services
**Goal**: Implement bulk operations services

**Actions**:
- [ ] Create ExportService for CSV/Excel generation
- [ ] Create ImportService for parsing files
- [ ] Add necessary dependencies (csv-parser, exceljs)

**Files to create**:
- `api/src/infrastructure/services/ExportService.ts`
- `api/src/infrastructure/services/ImportService.ts`

**Verification**:
```bash
cd api
npm install csv-parser exceljs
npm run build
```

---

### STEP 7: Implement Bulk Operations Use Cases
**Goal**: Create use cases for import/export

**Actions**:
- [ ] Create BulkExportProductsUseCase
- [ ] Create BulkImportProductsUseCase
- [ ] Add validation and error handling

**Files to create**:
- `api/src/application/usecases/product/BulkExportProductsUseCase.ts`
- `api/src/application/usecases/product/BulkImportProductsUseCase.ts`

**Verification**:
```bash
cd api
npm run build
npm run test
```

---

### STEP 8: Update Admin Product Controller
**Goal**: Add new endpoints to controller

**Actions**:
- [ ] Add listProductMasters method
- [ ] Add getProductMasterById method
- [ ] Add uploadProductImage method
- [ ] Add bulkExport method
- [ ] Add bulkImport method

**Files to modify**:
- `api/src/api/controllers/AdminProductController.ts`

**Verification**:
```bash
cd api
npm run build
npm run lint
```

---

### STEP 9: Create Validation Schemas
**Goal**: Add validation for new endpoints

**Actions**:
- [ ] Create schemas for list filters
- [ ] Create schemas for bulk operations
- [ ] Add file upload validation

**Files to create/modify**:
- `api/src/api/schemas/productSchemas.ts`

**Verification**:
```bash
cd api
npm run build
```

---

### STEP 10: Update Admin Product Routes
**Goal**: Add new routes for all endpoints

**Actions**:
- [ ] Add GET /masters route
- [ ] Add GET /masters/:id route
- [ ] Add POST /masters/upload-image route
- [ ] Add POST /masters/bulk-export route
- [ ] Add POST /masters/bulk-import route

**Files to modify**:
- `api/src/api/routes/adminProductRoutes.ts`

**Verification**:
```bash
cd api
npm run build
npm run dev
# Test endpoints with Postman/curl
```

---

### STEP 11: Register Dependencies in Container
**Goal**: Wire up all new services and use cases

**Actions**:
- [ ] Register S3Repository
- [ ] Register S3Service
- [ ] Register ExportService
- [ ] Register ImportService
- [ ] Register new use cases

**Files to modify**:
- `api/src/infrastructure/config/container.ts`

**Verification**:
```bash
cd api
npm run build
npm start
```

---

### STEP 12: Update Web-Admin Types
**Goal**: Add TypeScript types for frontend

**Actions**:
- [ ] Update product types with images
- [ ] Add bulk operation types
- [ ] Add filter types

**Files to modify**:
- `web-admin/src/shared/types/product.ts`
- `web-admin/src/shared/types/api.ts`

**Verification**:
```bash
cd web-admin
npm run type-check
```

---

### STEP 13: Create Image Upload Component
**Goal**: Build image management UI

**Actions**:
- [ ] Create ImageUploadZone atom
- [ ] Create ProductImageManager organism
- [ ] Integrate with S3 upload

**Files to create**:
- `web-admin/src/shared/components/atoms/ImageUploadZone/index.tsx`
- `web-admin/src/shared/components/organisms/ProductImageManager/index.tsx`

**Verification**:
```bash
cd web-admin
npm run build
npm run lint
```

---

### STEP 14: Create Filter Components
**Goal**: Build search and filter UI

**Actions**:
- [ ] Create ProductFilters molecule
- [ ] Add category dropdown
- [ ] Add product type filter

**Files to create**:
- `web-admin/src/shared/components/molecules/ProductFilters/index.tsx`

**Verification**:
```bash
cd web-admin
npm run build
```

---

### STEP 15: Create Bulk Operations Components
**Goal**: Build import/export UI

**Actions**:
- [ ] Create ProductBulkActions organism
- [ ] Add export dialog
- [ ] Add import dialog with preview

**Files to create**:
- `web-admin/src/shared/components/organisms/ProductBulkActions/index.tsx`

**Verification**:
```bash
cd web-admin
npm run build
```

---

### STEP 16: Update Admin Store
**Goal**: Add state management for new features

**Actions**:
- [ ] Add filter state
- [ ] Add language selection state
- [ ] Add bulk operation state
- [ ] Create new actions

**Files to modify**:
- `web-admin/src/shared/store/AdminStore.ts`

**Verification**:
```bash
cd web-admin
npm run type-check
```

---

### STEP 17: Update Admin Product Service
**Goal**: Add methods for new API endpoints

**Actions**:
- [ ] Update getProductMasters to use filters
- [ ] Add uploadImage method
- [ ] Add export/import methods

**Files to modify**:
- `web-admin/src/shared/services/AdminProductService.ts`

**Verification**:
```bash
cd web-admin
npm run build
```

---

### STEP 18: Update Products Page
**Goal**: Integrate all new components

**Actions**:
- [ ] Add filters component
- [ ] Add bulk actions component
- [ ] Update table to show images
- [ ] Add language switcher

**Files to modify**:
- `web-admin/src/app/products/page.tsx`

**Verification**:
```bash
cd web-admin
npm run dev
# Test all features in browser
```

---

### STEP 19: Add Translations
**Goal**: Add text for new features

**Actions**:
- [ ] Add filter labels
- [ ] Add bulk operation messages
- [ ] Add image upload text
- [ ] Add validation messages

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

### STEP 20: Final Integration Testing
**Goal**: Verify complete feature works end-to-end

**Actions**:
- [ ] Test product listing with filters
- [ ] Test image upload to S3
- [ ] Test bulk export (CSV and Excel)
- [ ] Test bulk import with validation
- [ ] Test permissions for different user roles
- [ ] Test language switching

**Verification**:
```bash
# Start all services
cd infrastructure
sudo docker compose up

# Run API tests
cd ../api
npm test

# Run web-admin
cd ../web-admin
npm run dev
```

---

## Post-Implementation Checklist

- [ ] All tests passing
- [ ] No TypeScript errors
- [ ] No ESLint warnings
- [ ] API documentation updated
- [ ] Environment variables documented
- [ ] S3 bucket configured with proper permissions
- [ ] Performance tested with large datasets
- [ ] Security review completed
- [ ] Monitoring configured

## Rollback Plan

If issues arise during deployment:

1. **Database**: Migration includes down() method
2. **API**: Previous version can be redeployed
3. **Frontend**: Feature flags can disable new functionality
4. **S3**: Images are isolated in separate bucket

## Success Criteria

- [ ] Products page loads in < 2 seconds
- [ ] Image uploads complete in < 5 seconds
- [ ] Bulk import handles 1000+ products
- [ ] All user roles have appropriate access
- [ ] No regression in existing functionality

## Notes

- Each step should be completed and verified before moving to the next
- Run build, type-check, and lint after each major change
- Commit changes after each successful step
- If blocked, check error logs and resolve before continuing