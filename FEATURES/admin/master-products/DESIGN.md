# TECHNICAL DESIGN - Master Products Management

## Overview
This document outlines the technical design for implementing the Master Products Management feature across the Museo platform. The implementation follows Clean Architecture principles and integrates with existing systems.

## Architecture Overview

### Systems Involved
1. **Web-Admin** (Next.js) - Frontend interface for product management
2. **API** (Node.js/Express) - Backend services following Clean Architecture
3. **Infrastructure** - PostgreSQL database with product schemas
4. **External Services** - Amazon S3 for image storage

## Database Design

### Schema Updates

#### product_masters table modification
```sql
-- Add images field to product_masters table
ALTER TABLE product_masters 
ADD COLUMN images JSONB;

-- Image structure:
-- [
--   {
--     "url": "https://s3.amazonaws.com/bucket/path/image.jpg",
--     "alt": "Product front view",
--     "isTile": true,
--     "isPrimary": false
--   }
-- ]
```

## API Design

### New Endpoints

#### 1. GET /api/v1/admin/products/masters
- **Purpose**: List product masters with pagination, search, and filters
- **Query Parameters**:
  - `page` (number): Page number (default: 1)
  - `limit` (number): Items per page (default: 20)
  - `search` (string): Search by name across all languages
  - `category` (string): Filter by category ID
  - `productType` (string): Filter by product type (physical/digital/service)
  - `status` (string): Filter by status
  - `sortBy` (string): Sort field (default: updatedAt)
  - `sortOrder` (string): Sort order asc/desc (default: desc)
- **Response**: 
```json
{
  "success": true,
  "data": {
    "items": [...],
    "total": 150,
    "page": 1,
    "limit": 20,
    "totalPages": 8
  }
}
```

#### 2. GET /api/v1/admin/products/masters/:id
- **Purpose**: Get single product master by ID
- **Response**: Complete product master object with all fields

#### 3. POST /api/v1/admin/products/masters/bulk-export
- **Purpose**: Export products to CSV/Excel
- **Request Body**:
```json
{
  "format": "csv" | "excel",
  "productIds": ["id1", "id2"] | null, // null = export all
  "fields": ["name", "sku", "status", ...] | null // null = all fields
}
```
- **Response**: File download

#### 4. POST /api/v1/admin/products/masters/bulk-import
- **Purpose**: Import products from CSV/Excel
- **Request**: Multipart form data with file
- **Query Parameters**:
  - `mode`: "replace" | "merge"
- **Response**: Import results with success/error details

#### 5. POST /api/v1/admin/products/masters/upload-image
- **Purpose**: Upload image to S3
- **Request**: Multipart form data with image file
- **Response**: 
```json
{
  "success": true,
  "data": {
    "url": "https://s3.amazonaws.com/museo-products/...",
    "size": 1024000,
    "mimetype": "image/jpeg"
  }
}
```

### Updated Controllers

#### AdminProductController
```typescript
// New methods to add:
- listProductMasters(req, res)
- getProductMasterById(req, res) 
- bulkExportProducts(req, res)
- bulkImportProducts(req, res)
- uploadProductImage(req, res)
```

### New Use Cases

#### 1. GetProductMastersUseCase
- **Purpose**: Retrieve paginated list of product masters
- **Dependencies**: ProductRepository
- **Features**:
  - Multi-language search
  - Category and type filtering
  - Sorting and pagination

#### 2. GetProductMasterByIdUseCase
- **Purpose**: Retrieve single product master
- **Dependencies**: ProductRepository
- **Features**:
  - Include all related data
  - Permission checking

#### 3. BulkExportProductsUseCase
- **Purpose**: Export products to file
- **Dependencies**: ProductRepository, ExportService
- **Features**:
  - Support CSV and Excel formats
  - Selective field export
  - Multi-language support

#### 4. BulkImportProductsUseCase
- **Purpose**: Import products from file
- **Dependencies**: ProductRepository, ImportService
- **Features**:
  - Validation
  - Replace/merge modes
  - Transaction support
  - Error reporting

#### 5. UploadProductImageUseCase
- **Purpose**: Upload image to S3
- **Dependencies**: S3Repository
- **Features**:
  - File validation
  - S3 upload
  - URL generation

### New Services

#### S3Service
```typescript
interface S3Service {
  uploadImage(file: Buffer, filename: string): Promise<string>
  deleteImage(url: string): Promise<void>
  getSignedUrl(key: string): Promise<string>
}
```

#### ExportService
```typescript
interface ExportService {
  exportToCSV(products: ProductMaster[]): Buffer
  exportToExcel(products: ProductMaster[]): Buffer
}
```

#### ImportService
```typescript
interface ImportService {
  parseCSV(buffer: Buffer): Promise<ParsedProduct[]>
  parseExcel(buffer: Buffer): Promise<ParsedProduct[]>
  validateProducts(products: ParsedProduct[]): ValidationResult
}
```

### Repository Updates

#### ProductRepository
```typescript
// New methods:
interface ProductRepository {
  // Existing methods...
  
  findMasters(params: {
    page: number
    limit: number
    search?: string
    category?: string
    productType?: string
    status?: string
    sortBy?: string
    sortOrder?: 'asc' | 'desc'
  }): Promise<PaginatedResult<ProductMaster>>
  
  findMasterById(id: string): Promise<ProductMaster | null>
  
  bulkCreate(products: CreateProductMasterRequest[]): Promise<ProductMaster[]>
  
  bulkUpdate(products: UpdateProductMasterRequest[]): Promise<ProductMaster[]>
}
```

#### New S3Repository
```typescript
interface S3Repository {
  upload(params: {
    bucket: string
    key: string
    body: Buffer
    contentType: string
  }): Promise<S3UploadResult>
  
  delete(params: {
    bucket: string
    key: string
  }): Promise<void>
  
  getSignedUrl(params: {
    bucket: string
    key: string
    expires: number
  }): Promise<string>
}
```

## Web-Admin Design

### Component Structure

#### Pages
- `/products/page.tsx` - Update to use new endpoints

#### New Components

##### organisms/ProductImageManager
- Drag-drop upload interface
- Image gallery management
- Tile/Primary image selection
- S3 upload integration

##### organisms/ProductBulkActions
- Export dialog with options
- Import dialog with file upload
- Progress indicators

##### molecules/ProductFilters
- Category dropdown (hierarchical)
- Product type filter
- Status filter
- Clear filters button

##### molecules/LanguageSwitcher
- Language selection dropdown
- Visual indicators for translation status

##### atoms/ImageUploadZone
- Drag-drop area
- File validation
- Upload progress

### State Management Updates

#### AdminStore
```typescript
interface AdminStore {
  // Existing state...
  
  // New state
  productFilters: {
    search: string
    category: string
    productType: string
    status: string
  }
  
  selectedLanguage: string
  
  bulkOperation: {
    type: 'import' | 'export' | null
    inProgress: boolean
    progress: number
  }
  
  // New actions
  setProductFilters: (filters: Partial<ProductFilters>) => void
  setSelectedLanguage: (language: string) => void
  startBulkOperation: (type: 'import' | 'export') => void
  updateBulkProgress: (progress: number) => void
  completeBulkOperation: () => void
}
```

### Service Updates

#### AdminProductService
```typescript
// New methods:
class AdminProductService extends BaseAPIService {
  // Existing methods...
  
  async uploadImage(file: File): Promise<Result<ImageUploadResult>>
  
  async exportProducts(params: ExportParams): Promise<Blob>
  
  async importProducts(file: File, mode: 'replace' | 'merge'): Promise<Result<ImportResult>>
  
  async getCategories(): Promise<Result<Category[]>>
}
```

## Integration Points

### Amazon S3 Configuration
```typescript
// Environment variables needed:
AWS_ACCESS_KEY_ID=xxx
AWS_SECRET_ACCESS_KEY=xxx
AWS_REGION=us-east-1
AWS_S3_BUCKET=museo-products
```

### Image Upload Flow
1. User selects image in web-admin
2. Frontend validates file (type, size)
3. Frontend calls upload endpoint
4. API validates and uploads to S3
5. API returns S3 URL
6. Frontend adds URL to product images array
7. Save product with updated images

### Import/Export Flow

#### Export:
1. User selects export options
2. API queries products based on filters
3. Generate CSV/Excel file in memory
4. Stream file to user for download

#### Import:
1. User uploads CSV/Excel file
2. API parses and validates file
3. Show preview with validation errors
4. User confirms import
5. Process import in transaction
6. Return results summary

## Security Considerations

### Permissions
- Check user permissions for all operations
- Admin: Full access
- Manager: View, Update, Export only
- Operator: View and Export only

### File Upload Security
- Validate file types (images: jpg, png, webp)
- Maximum file size: 5MB
- Virus scanning (if available)
- Generate unique S3 keys to prevent overwrites

### S3 Security
- Use IAM roles with minimal permissions
- Private bucket with signed URLs
- Set appropriate CORS policies
- Enable S3 versioning for backup

## Performance Considerations

### Database
- Use indexes for search fields
- Optimize JSONB queries for multi-language search
- Use database views for complex queries
- Implement query result caching

### API
- Implement pagination for all list endpoints
- Use streaming for large exports
- Process imports in batches
- Add request rate limiting

### Frontend
- Lazy load images
- Implement virtual scrolling for large lists
- Debounce search inputs
- Cache category and brand lists

## Error Handling

### API Errors
- Consistent error response format
- Detailed validation messages
- Transaction rollback on failures
- Comprehensive logging

### Frontend Errors
- User-friendly error messages
- Retry mechanisms for network failures
- Form validation feedback
- Progress indicators for long operations

## Testing Strategy

### Unit Tests
- Use case tests with mocked repositories
- Repository tests with test database
- Service tests with mocked dependencies

### Integration Tests
- API endpoint tests
- S3 upload/download tests
- Import/export functionality
- Permission checking

### E2E Tests
- Complete product CRUD flow
- Image upload and management
- Bulk import/export operations
- Search and filter functionality

## Migration Strategy

### Database Migration
1. Create new migration file for images column
2. Run migration in staging first
3. Verify no issues
4. Run in production during maintenance window

### API Deployment
1. Deploy new endpoints
2. Maintain backward compatibility
3. Monitor for errors
4. Enable features gradually

### Frontend Deployment
1. Deploy with feature flags
2. Test with limited users
3. Monitor performance
4. Full rollout after validation

## Monitoring and Logging

### Metrics to Track
- API response times
- Image upload success/failure rates
- Import/export completion rates
- Search query performance
- S3 storage usage

### Logging
- All CRUD operations
- Permission denials
- Import/export activities
- Error details
- Performance bottlenecks