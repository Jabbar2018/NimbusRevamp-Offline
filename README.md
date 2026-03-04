# NimbusRMSCoreOffline - API Service for IndexedDB Synchronization

A .NET 8.0 Web API service that provides RESTful endpoints for synchronizing data between a SQL Server database and client-side IndexedDB. This backend service is designed to support offline-first functionality in frontend applications by providing efficient data synchronization APIs.

## 🎯 Project Purpose

This API service is designed to:
- **Provide data synchronization endpoints** for frontend applications
- **Support offline-first architecture** by enabling IndexedDB creation and updates
- **Handle incremental and full data syncs** based on timestamps
- **Return structured data** ready for IndexedDB storage
- **Support both bulk and individual entity synchronization**

## 🏗️ Architecture Overview

```
┌─────────────────────┐         ┌──────────────────┐         ┌─────────────────┐
│   SQL Server DB     │ ◄─────► │  NimbusRMSCore  │ ◄─────► │  Frontend App   │
│   (Source Data)     │  ADO.NET │  Offline API    │  HTTP   │  (IndexedDB)    │
└─────────────────────┘         └──────────────────┘         └─────────────────┘
```

### Data Flow
1. **Frontend Request** → HTTP POST to API endpoint
2. **API Processing** → Execute stored procedures with filters
3. **Data Retrieval** → Fetch from SQL Server database
4. **Response Formation** → Structure data for IndexedDB
5. **Frontend Storage** → Store in browser IndexedDB

## 🛠️ Technology Stack

- **.NET 8.0** - Framework
- **ASP.NET Core Web API** - API framework
- **ADO.NET** - Database access (via stored procedures)
- **JWT Bearer Authentication** - Security
- **Swagger/OpenAPI** - API documentation
- **AutoMapper** - Object mapping
- **CORS** - Cross-origin resource sharing

## 📁 Project Structure

```
NimbusRMSCoreOffline/
├── NimbusRevampOffline/              # Main API project
│   ├── Controllers/
│   │   └── Offline/
│   │       └── OfflineController.cs  # Main API controller
│   ├── Program.cs                    # Application entry point
│   └── appsettings.json              # Configuration
├── NimbusRevamp.Services/            # Business logic layer
│   └── Offline/
│       ├── IOfflineService.cs        # Service interface
│       └── OfflineService.cs         # Service implementation
├── NimbusRevamp.Repositories/        # Data access layer
│   └── Offline/
│       ├── IOfflineRepository.cs     # Repository interface
│       └── OfflineRepository.cs      # Repository implementation
└── NimbusRevamp.DTOs/                # Data transfer objects
    ├── RequestDTOs/
    │   └── Offline/                  # Request DTOs
    └── ResponseDTOs/
        └── Offline/                  # Response DTOs
```

## 🚀 Getting Started

### Prerequisites

- **.NET 8.0 SDK** or higher
- **SQL Server** (with database configured)
- **Visual Studio 2022** or **VS Code** (recommended)
- **Postman** or similar API testing tool (optional)

### Installation

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd NimbusRMSCoreOffline
   ```

2. **Restore NuGet packages**
   ```bash
   dotnet restore
   ```

3. **Configure database connection**
   
   Edit `NimbusRevampOffline/appsettings.json`:
   ```json
   {
     "ConnectionStrings": {
       "DefaultConnection": "Server=YOUR_SERVER;Database=YOUR_DB;User Id=YOUR_USER;Password=YOUR_PASSWORD;TrustServerCertificate=True;",
       "ReadyOnlyConnection": "Server=YOUR_SERVER;Database=YOUR_DB;User Id=YOUR_USER;Password=YOUR_PASSWORD;TrustServerCertificate=True;"
     }
   }
   ```

4. **Configure JWT settings** (if needed)
   ```json
   {
     "Jwt": {
       "Key": "YOUR_SECRET_KEY",
       "Issuer": "NimbusRevamp",
       "Audience": "NimbusRevampUsers",
       "ExpiryInMinutes": 15
     }
   }
   ```

5. **Run the application**
   ```bash
   cd NimbusRevampOffline
   dotnet run
   ```

6. **Access Swagger UI**
   - Navigate to: `https://localhost:5001/swagger` (or your configured port)
   - API documentation will be available

## 📡 API Endpoints

### Base URL
```
https://your-api-url/api/Offline
```

### Authentication
All endpoints require JWT Bearer authentication. Include the token in the Authorization header:
```
Authorization: Bearer <your-jwt-token>
```

---

### 1. Main DataSync API

**Endpoint**: `POST /api/Offline/DataSync`

**Purpose**: Synchronize all master data tables in a single request

**Request Body**:
```json
{
  "companyId": 1,
  "initialDataSyncing": false,
  "lastDataSyncTime": "2024-01-01T00:00:00Z"
}
```

**Request Parameters**:
- `companyId` (int, required): Company identifier
- `initialDataSyncing` (bool, optional): If true, fetches all data regardless of timestamp
- `lastDataSyncTime` (DateTime?, optional): Only fetch records modified after this time

**Response Structure**:
```json
{
  "success": true,
  "message": "Offline Data List retrieved successfully",
  "data": {
    "update": {
      "shop": [...],
      "suppliers": [...],
      "category": [...],
      "rcmsConfiguration": [...],
      "creditCards": [...],
      // ... 50+ entity types
    },
    "delete": {
      "shop": [...],
      "suppliers": [...],
      // ... deleted record IDs
    }
  }
}
```

**Example Request**:
```bash
curl -X POST "https://api.example.com/api/Offline/DataSync" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "companyId": 1,
    "lastDataSyncTime": "2024-12-01T00:00:00Z"
  }'
```

---

### 2. Product DataSync API

**Endpoint**: `POST /api/Offline/DataSync/Product`

**Purpose**: Synchronize product data separately (supports pagination)

**Request Body**:
```json
{
  "companyId": 1,
  "lastDataSyncTime": "2024-01-01T00:00:00Z",
  "packagingBarcode": false,
  "startIndex": 0
}
```

**Request Parameters**:
- `companyId` (int, required): Company identifier
- `lastDataSyncTime` (DateTime?, optional): Only fetch products modified after this time
- `packagingBarcode` (bool?, optional): Include packaging barcodes
- `startIndex` (int, required): Pagination start index

**Response Structure**:
```json
{
  "success": true,
  "message": "Offline Product Data List retrieved successfully",
  "data": {
    "update": {
      "product": [...]
    },
    "delete": {
      "product": [...],
      "productItem": [...],
      "productNestedBarcode": [...]
    }
  }
}
```

---

### 3. Individual Entity APIs (Paired - SELECT + DELETE)

These endpoints return both updated records and deleted record IDs for a specific entity type.

**Endpoint Pattern**: `POST /api/Offline/{EntityName}`

**Request Body**:
```json
{
  "companyId": 1,
  "lastDataSyncTime": "2024-01-01T00:00:00Z"
}
```

**Available Endpoints** (42 total):

| Endpoint | Entity Type | Description |
|----------|-------------|-------------|
| `POST /api/Offline/Shop` | Shop | Store/shop information |
| `POST /api/Offline/Supplier` | Supplier | Supplier master data |
| `POST /api/Offline/CreditCards` | CreditCards | Credit card configurations |
| `POST /api/Offline/LineItem` | LineItem | Line item definitions |
| `POST /api/Offline/Category` | Category | Product categories |
| `POST /api/Offline/ProductAssembly` | ProductAssembly | Product assembly data |
| `POST /api/Offline/Register` | Register | POS register information |
| `POST /api/Offline/EmployeeType` | EmployeeType | Employee type definitions |
| `POST /api/Offline/AccountHead` | AccountHead | Accounting account heads |
| `POST /api/Offline/MemberInfo` | MemberInfo | Member/customer information |
| `POST /api/Offline/Discount` | Discount | Discount configurations |
| `POST /api/Offline/SecurityGroup` | SecurityGroup | Security group definitions |
| `POST /api/Offline/PosCashManagement` | PosCashManagement | Cash management settings |
| `POST /api/Offline/Color` | Color | Color master data |
| `POST /api/Offline/ProductSize` | ProductSize | Product size definitions |
| `POST /api/Offline/SerialNumberDetail` | SerialNumberDetail | Serial number tracking |
| `POST /api/Offline/StoreBasedPrices` | StoreBasedPrices | Store-specific pricing |
| `POST /api/Offline/CustomerTypeBasedPrices` | CustomerTypeBasedPrices | Customer type pricing |
| `POST /api/Offline/ProductGroup` | ProductGroup | Product grouping |
| `POST /api/Offline/SubCategory` | SubCategory | Product subcategories |
| `POST /api/Offline/ProductAttribute1` | ProductAttribute1 | Product attribute 1 |
| `POST /api/Offline/ProductAttribute3` | ProductAttribute3 | Product attribute 3 |
| `POST /api/Offline/IBattribute2` | IBattribute2 | Inventory attribute 2 |
| `POST /api/Offline/IBattribute4` | IBattribute4 | Inventory attribute 4 |
| `POST /api/Offline/IBattribute5` | IBattribute5 | Inventory attribute 5 |
| `POST /api/Offline/IBattribute6` | IBattribute6 | Inventory attribute 6 |
| `POST /api/Offline/IBattribute7` | IBattribute7 | Inventory attribute 7 |
| `POST /api/Offline/IBattribute8` | IBattribute8 | Inventory attribute 8 |
| `POST /api/Offline/IBattribute9` | IBattribute9 | Inventory attribute 9 |
| `POST /api/Offline/Kitchen` | Kitchen | Kitchen display settings |
| `POST /api/Offline/DisplayGroup` | DisplayGroup | Display group definitions |
| `POST /api/Offline/DisplayGroupItems` | DisplayGroupItems | Display group items |
| `POST /api/Offline/KDSConfiguration` | KDSConfiguration | Kitchen display system config |
| `POST /api/Offline/BlockMenuItems` | BlockMenuItems | Blocked menu items |
| `POST /api/Offline/TableGroups` | TableGroups | Table group definitions |
| `POST /api/Offline/Tables` | Tables | Table master data |
| `POST /api/Offline/ItemModifiers` | ItemModifiers | Item modifier definitions |
| `POST /api/Offline/MemberTypes` | MemberTypes | Member type definitions |
| `POST /api/Offline/MemberTypeDiscount` | MemberTypeDiscount | Member type discounts |
| `POST /api/Offline/DealsDetail` | DealsDetail | Deal configurations |
| `POST /api/Offline/ShopCreditCards` | ShopCreditCards | Shop credit card settings |
| `POST /api/Offline/DeliveryMan` | DeliveryMan | Delivery personnel data |

**Response Structure**:
```json
{
  "success": true,
  "message": "{Entity} data retrieved successfully",
  "data": {
    "update": [...],
    "delete": [...]
  }
}
```

---

### 4. Individual Entity APIs (SELECT-only)

These endpoints return only updated records (no delete tracking).

**Endpoint Pattern**: `POST /api/Offline/{EntityName}`

**Request Body**:
```json
{
  "companyId": 1,
  "lastDataSyncTime": "2024-01-01T00:00:00Z"
}
```

**Available Endpoints** (8 total):

| Endpoint | Entity Type | Description |
|----------|-------------|-------------|
| `POST /api/Offline/RCMSConfiguration` | RCMSConfiguration | System configuration settings |
| `POST /api/Offline/SecurityUsers` | SecurityUsers | User accounts |
| `POST /api/Offline/UserPrintSettings` | UserPrintSettings | User print preferences |
| `POST /api/Offline/CustomReceiptConfiguration` | CustomReceiptConfiguration | Receipt templates |
| `POST /api/Offline/ShopConfiguration` | ShopConfiguration | Shop-specific settings |
| `POST /api/Offline/ShopEmployees` | ShopEmployees | Employee assignments |
| `POST /api/Offline/GiftCardsDetail` | GiftCardsDetail | Gift card information |
| `POST /api/Offline/CboTableCollection` | CboTableCollection | Combo table collections |

**Response Structure**:
```json
{
  "success": true,
  "message": "{Entity} data retrieved successfully",
  "data": {
    "update": [...]
  }
}
```

---

## 🔄 Synchronization Strategies

### 1. Initial Sync (Full Data Load)
Use when setting up IndexedDB for the first time:
```json
{
  "companyId": 1,
  "initialDataSyncing": true
}
```

### 2. Incremental Sync (Delta Updates)
Use for regular synchronization to fetch only changes:
```json
{
  "companyId": 1,
  "lastDataSyncTime": "2024-12-15T10:30:00Z"
}
```

### 3. Full Resync (From Date)
Use to resync all data from a specific date:
```json
{
  "companyId": 1,
  "lastDataSyncTime": "1990-01-01T00:00:00Z"
}
```

## 📊 Response Data Structure

### Main DataSync Response
The main DataSync endpoint returns data for **50+ entity types** organized into `update` and `delete` sections:

**Update Section** includes:
- Shop, Suppliers, RCMSConfiguration, CreditCards, Department, Category
- ProductAssembly, ShopEmployee, Register, EmployeeType, AccountHead
- MemberInfo, SecurityUser, MemberTypes, CustomerTypeBasedPrice
- ProductPriceShopBased, Discount, SecurityGroup, UserPrintSettings
- CustomReceiptConfiguration, POSCashManagement, Color, ProductSize
- ProductGroup, SubCategory, ProductAttribute1, ProductAttribute3
- SerialNumberDetail, CboTableCollection, ShopConfiguration
- ProductAttribute2-9, Kitchen, DisplayGroup, DisplayGroupItem
- KDSConfiguration, BlockMenuItems, TableGroups, Tables
- ItemModifiers, MemberTypeDiscount, DealsDetail, ShopCreditCards
- GiftCardsDetail, DeliveryMan

**Delete Section** includes:
- IDs of records that have been deleted (for tables that support delete tracking)

## 🔐 Authentication & Security

### JWT Configuration
The API uses JWT Bearer authentication. Configure in `appsettings.json`:
```json
{
  "Jwt": {
    "Key": "your-secret-key-here",
    "Issuer": "NimbusRevamp",
    "Audience": "NimbusRevampUsers",
    "ExpiryInMinutes": 15
  }
}
```

### CORS Configuration
CORS is configured to allow all origins in development. For production, update `Program.cs`:
```csharp
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowSpecificOrigins",
        builder => builder
            .WithOrigins("https://your-frontend-domain.com")
            .AllowAnyMethod()
            .AllowAnyHeader());
});
```

## ⚙️ Configuration

### Database Connection Strings
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=...;Database=...;User Id=...;Password=...;TrustServerCertificate=True;",
    "ReadyOnlyConnection": "Server=...;Database=...;User Id=...;Password=...;TrustServerCertificate=True;"
  }
}
```

### Command Timeout
```json
{
  "CommandTimeoutSeconds": 300
}
```

## 🧪 Testing the API

### Using Swagger UI
1. Run the application
2. Navigate to `/swagger`
3. Click "Authorize" and enter your JWT token
4. Test endpoints directly from the UI

### Using Postman
1. Create a new POST request
2. Set URL: `https://your-api/api/Offline/DataSync`
3. Add header: `Authorization: Bearer YOUR_TOKEN`
4. Set body (JSON):
   ```json
   {
     "companyId": 1,
     "lastDataSyncTime": "2024-01-01T00:00:00Z"
   }
   ```
5. Send request

### Using cURL
```bash
curl -X POST "https://api.example.com/api/Offline/DataSync" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "companyId": 1,
    "lastDataSyncTime": "2024-12-01T00:00:00Z"
  }'
```

## 📝 Frontend Integration Example

### JavaScript/TypeScript Example
```javascript
// Main DataSync
async function syncData() {
  const response = await fetch('https://api.example.com/api/Offline/DataSync', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      companyId: 1,
      lastDataSyncTime: new Date('2024-12-01').toISOString()
    })
  });
  
  const result = await response.json();
  
  if (result.success) {
    // Store in IndexedDB
    await syncUpdateDataIndexDb(result.data.update);
    await deleteDataIndexDb(result.data.delete);
  }
}

// Individual Entity Sync
async function syncShop() {
  const response = await fetch('https://api.example.com/api/Offline/Shop', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      companyId: 1,
      lastDataSyncTime: new Date('2024-12-01').toISOString()
    })
  });
  
  const result = await response.json();
  // Process result.data.update and result.data.delete
}
```

## 🐛 Troubleshooting

### Common Issues

1. **401 Unauthorized**
   - Verify JWT token is valid and not expired
   - Check token is included in Authorization header
   - Verify JWT configuration in appsettings.json

2. **500 Internal Server Error**
   - Check database connection string
   - Verify stored procedures exist in database
   - Check application logs for detailed errors

3. **CORS Errors**
   - Verify CORS policy allows your frontend origin
   - Check CORS middleware is configured correctly

4. **Timeout Errors**
   - Increase `CommandTimeoutSeconds` in appsettings.json
   - Optimize stored procedures for better performance
   - Consider pagination for large datasets

5. **Empty Response**
   - Verify `companyId` is correct
   - Check if data exists in database for the given filters
   - Verify `lastDataSyncTime` is in correct format

## 📈 Performance Considerations

- **Use Incremental Syncs**: Always use `lastDataSyncTime` for regular syncs to minimize data transfer
- **Pagination for Products**: Use `startIndex` parameter for product sync to handle large datasets
- **Individual APIs**: Use specific entity APIs when you only need certain data types
- **Connection Pooling**: Database connections are pooled automatically
- **Async Operations**: All endpoints are async for better scalability

## 🔧 Development

### Building the Project
```bash
dotnet build
```

### Running Tests
```bash
dotnet test
```

### Publishing
```bash
dotnet publish -c Release -o ./publish
```

## 📚 API Documentation

Full API documentation is available via Swagger UI when running the application:
- Development: `https://localhost:5001/swagger`
- Production: `https://your-api-url/swagger`

## 🤝 Contributing

When adding new endpoints:

1. Add method to `IOfflineService` interface
2. Implement in `OfflineService` class
3. Add repository method in `IOfflineRepository`
4. Implement repository method
5. Add controller endpoint in `OfflineController`
6. Update DTOs if needed
7. Test thoroughly
8. Update this README

## 📄 License

This project is proprietary software. All rights reserved.

## 👥 Team

- **Project**: Nimbus Cloud Retail Software
- **Lead Developer**: Muhammad Muhib Mirza

## 📞 Support

For API-related questions:
- Check Swagger documentation at `/swagger`
- Review controller implementations in `NimbusRevampOffline/Controllers/Offline/`
- Check service layer in `NimbusRevamp.Services/Offline/`
- Review repository layer in `NimbusRevamp.Repositories/Offline/`

---

**Last Updated**: December 2024  
**Version**: 1.0.0  
**Status**: Active Development  
**Framework**: .NET 8.0
