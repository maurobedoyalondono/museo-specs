# TECHNICAL DESIGN - Site Settings Management

## Overview

This document outlines the technical design for implementing a comprehensive settings management system following Museo's clean architecture principles.

## Database Design

### Schema File: `10_settings.sql`

Location: `/home/negrito/src/projects/Museo/infrastructure/schemas/db/10_settings.sql`

```sql
-- Setting Groups
CREATE TABLE setting_groups (
    id VARCHAR(50) PRIMARY KEY,
    name JSONB NOT NULL, -- {"en": "System", "es": "Sistema"}
    description JSONB,
    icon VARCHAR(100),
    sort_order INTEGER DEFAULT 0,
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Settings Configuration
CREATE TABLE settings (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    group_id VARCHAR(50) NOT NULL REFERENCES setting_groups(id) ON DELETE RESTRICT,
    key VARCHAR(100) NOT NULL,
    name JSONB NOT NULL, -- {"en": "Site Name", "es": "Nombre del Sitio"}
    description JSONB,
    type VARCHAR(20) NOT NULL CHECK (type IN (
        'text', 'url', 'encrypted_text', 'email', 'domain',
        'boolean', 'number', 'select', 'json', 'color',
        'file', 'textarea', 'date', 'time', 'array'
    )),
    value TEXT, -- Actual value (encrypted if type is encrypted_text)
    validation_rules JSONB, -- {"required": true, "min": 1, "max": 100, "pattern": "^[A-Z]"}
    options JSONB, -- For select type: [{"value": "UTC", "label": {"en": "UTC"}}]
    is_required BOOLEAN DEFAULT false,
    sort_order INTEGER DEFAULT 0,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(group_id, key)
);

-- Setting History (Audit Log)
CREATE TABLE setting_history (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    setting_id UUID NOT NULL REFERENCES settings(id) ON DELETE CASCADE,
    group_id VARCHAR(50) NOT NULL,
    key VARCHAR(100) NOT NULL,
    old_value TEXT,
    new_value TEXT,
    changed_by VARCHAR(100) NOT NULL, -- User ID who made the change
    changed_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    ip_address VARCHAR(45),
    user_agent TEXT
);

-- Indexes
CREATE INDEX idx_settings_group_id ON settings(group_id);
CREATE INDEX idx_settings_key ON settings(key);
CREATE INDEX idx_settings_group_key ON settings(group_id, key);
CREATE INDEX idx_setting_history_setting_id ON setting_history(setting_id);
CREATE INDEX idx_setting_history_changed_at ON setting_history(changed_at);

-- Triggers
CREATE TRIGGER update_settings_updated_at
    BEFORE UPDATE ON settings
    FOR EACH ROW
    EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_setting_groups_updated_at
    BEFORE UPDATE ON setting_groups
    FOR EACH ROW
    EXECUTE FUNCTION update_updated_at_column();
```

### Migrations

1. **Schema Migration**: `[timestamp]_create_settings_tables.ts`
   - Creates the tables defined in the schema

2. **Data Migration**: `[timestamp]_seed_settings_data.ts`
   - Inserts setting groups
   - Inserts initial System settings

## API Design

### Domain Layer

#### Entities

**SettingGroup** (`domain/entities/SettingGroup.ts`)
```typescript
interface SettingGroup {
  id: string;
  name: LocalizedString;
  description?: LocalizedString;
  icon?: string;
  sortOrder: number;
  isActive: boolean;
  createdAt: Date;
  updatedAt: Date;
}
```

**Setting** (`domain/entities/Setting.ts`)
```typescript
interface Setting {
  id: string;
  groupId: string;
  key: string;
  name: LocalizedString;
  description?: LocalizedString;
  type: SettingType;
  value: any;
  validationRules?: ValidationRules;
  options?: SelectOption[];
  isRequired: boolean;
  sortOrder: number;
  createdAt: Date;
  updatedAt: Date;
}

type SettingType = 
  | 'text' | 'url' | 'encrypted_text' | 'email' | 'domain'
  | 'boolean' | 'number' | 'select' | 'json' | 'color'
  | 'file' | 'textarea' | 'date' | 'time' | 'array';

interface ValidationRules {
  required?: boolean;
  min?: number;
  max?: number;
  pattern?: string;
  minLength?: number;
  maxLength?: number;
}
```

**SettingHistory** (`domain/entities/SettingHistory.ts`)
```typescript
interface SettingHistory {
  id: string;
  settingId: string;
  groupId: string;
  key: string;
  oldValue: any;
  newValue: any;
  changedBy: string;
  changedAt: Date;
  ipAddress?: string;
  userAgent?: string;
}
```

#### Repository Interfaces

**ISettingsRepository** (`domain/repositories/ISettingsRepository.ts`)
```typescript
interface ISettingsRepository {
  // Groups
  getAllGroups(): Promise<SettingGroup[]>;
  getGroupById(id: string): Promise<SettingGroup | null>;
  
  // Settings
  getSettingsByGroup(groupId: string): Promise<Setting[]>;
  getSettingByKey(groupId: string, key: string): Promise<Setting | null>;
  getAllSettings(): Promise<Setting[]>;
  updateSetting(groupId: string, key: string, value: any, userId: string, meta?: AuditMeta): Promise<Setting>;
  bulkUpdateSettings(updates: SettingUpdate[], userId: string, meta?: AuditMeta): Promise<Setting[]>;
  
  // History
  getSettingHistory(settingId: string, limit?: number): Promise<SettingHistory[]>;
  getHistoryByDateRange(startDate: Date, endDate: Date): Promise<SettingHistory[]>;
}

interface SettingUpdate {
  groupId: string;
  key: string;
  value: any;
}

interface AuditMeta {
  ipAddress?: string;
  userAgent?: string;
}
```

### Application Layer

#### Use Cases

1. **GetSettingsByGroupUseCase**
2. **GetSettingByKeyUseCase**
3. **UpdateSettingUseCase**
4. **BulkUpdateSettingsUseCase**
5. **ExportSettingsUseCase**
6. **ImportSettingsUseCase**
7. **GetSettingHistoryUseCase**
8. **ValidateSettingUseCase**

#### Services

**SettingsService** (`application/services/SettingsService.ts`)
```typescript
class SettingsService {
  constructor(
    private settingsRepository: ISettingsRepository,
    private cacheService: ICacheService,
    private encryptionService: IEncryptionService
  ) {}

  async getSettingsByGroup(groupId: string): Promise<Setting[]> {
    // Check cache first
    // Decrypt encrypted_text values
    // Return settings
  }

  async getSettingByKey(groupId: string, key: string): Promise<any> {
    // Get from cache or repository
    // Decrypt if needed
    // Return typed value
  }

  async invalidateCache(): Promise<void> {
    // Clear settings cache
  }
}
```

### Infrastructure Layer

#### Repository Implementation

**PostgresSettingsRepository** (`infrastructure/repositories/PostgresSettingsRepository.ts`)
- Implements ISettingsRepository
- Handles encryption/decryption for encrypted_text type
- Manages database transactions

#### Services

**SettingValidationService** (`infrastructure/services/SettingValidationService.ts`)
- Validates setting values based on type and rules
- Returns validation errors

**SettingExportService** (`infrastructure/services/SettingExportService.ts`)
- Exports settings to JSON/XML
- Handles sensitive data exclusion

**SettingImportService** (`infrastructure/services/SettingImportService.ts`)
- Imports settings from JSON/XML
- Validates before applying

### API Layer

#### Controllers

**AdminSettingsController** (`api/controllers/AdminSettingsController.ts`)
- List groups
- Get settings by group
- Update setting
- Bulk update
- Export/Import
- View history

#### Routes

**adminSettingsRoutes.ts** (`api/routes/adminSettingsRoutes.ts`)
```typescript
GET    /api/v1/admin/settings/groups
GET    /api/v1/admin/settings/groups/:groupId
PUT    /api/v1/admin/settings/groups/:groupId/settings/:key
POST   /api/v1/admin/settings/bulk-update
GET    /api/v1/admin/settings/export
POST   /api/v1/admin/settings/import
GET    /api/v1/admin/settings/history
GET    /api/v1/admin/settings/:settingId/history
```

#### Validation Schemas

**settingSchemas.ts** (`api/schemas/settingSchemas.ts`)
- Update setting schema
- Bulk update schema
- Import schema
- Export query schema

## Web Admin UI Design

### Components Structure

#### Atoms
- **SettingInput**: Dynamic input based on setting type
- **SettingLabel**: Label with help text
- **SettingValidationMessage**: Validation error display

#### Molecules
- **SettingField**: Complete field with label, input, and validation
- **SettingGroupCard**: Card for grouping settings
- **SettingHistoryItem**: Single history entry

#### Organisms
- **SettingsForm**: Form for a group of settings
- **SettingsImportDialog**: Import settings dialog
- **SettingsExportDialog**: Export settings dialog
- **SettingHistoryModal**: View setting history

### Pages

**Settings Page** (`app/settings/page.tsx`)
- List all setting groups
- Navigate to group settings

**Settings Group Page** (`app/settings/[groupId]/page.tsx`)
- Display all settings for a group
- Edit settings inline
- Save changes
- View history

### Services

**AdminSettingsService** (`shared/services/AdminSettingsService.ts`)
- Get groups
- Get settings by group
- Update setting
- Bulk update
- Export/Import
- Get history

### Store

**Update AdminStore** to include:
- Settings state
- Current group
- Pending changes
- History data

## Security Considerations

1. **Encryption**: All encrypted_text values use AES-256 encryption
2. **Validation**: Server-side validation for all setting updates
3. **Audit Trail**: Complete history of all changes
4. **Access Control**: Admin-only endpoints (middleware to be added later)
5. **Sensitive Data**: Encrypted values not included in exports by default

## Performance Considerations

1. **Caching**: All settings cached in Redis with automatic invalidation
2. **Lazy Loading**: Settings loaded by group, not all at once
3. **Bulk Operations**: Optimized for updating multiple settings
4. **Database Indexes**: Proper indexes for fast lookups

## Integration Points

1. **SettingsService**: Available via DI container for use throughout API
2. **Cache Invalidation**: Automatic on updates
3. **Event System**: Setting change events for real-time updates
4. **S3 Integration**: For file-type settings

## Error Handling

1. **Validation Errors**: Clear messages for invalid values
2. **Database Errors**: Proper error propagation
3. **Import Errors**: Detailed error report for failed imports
4. **Encryption Errors**: Graceful handling of encryption failures