# System Architecture Documentation

This document provides a comprehensive overview of the mini-rag system architecture, including high-level system design and detailed API flow diagrams.

## Table of Contents

1. [High-Level System Architecture](#high-level-system-architecture)
2. [API Flow Diagrams](#api-flow-diagrams)
   - [Welcome Endpoint](#1-welcome-endpoint)
   - [Upload Data Endpoint](#2-upload-data-endpoint)
   - [Process Data Endpoint](#3-process-data-endpoint)
3. [Assumptions](#assumptions)

---

## High-Level System Architecture

The mini-rag system follows a **Model-View-Controller (MVC)** architecture pattern, built on FastAPI framework. The system is designed to handle document uploads, processing, and chunking for Retrieval-Augmented Generation (RAG) applications.

```mermaid
graph TB
    subgraph ClientLayer[" "]
        direction TB
        Client[Client Application]
    end
    
    subgraph APILayer["API Layer (FastAPI)"]
        direction TB
        Router[API Routes]
        BaseRouter[Base Router<br/>/api/v1/]
        DataRouter[Data Router<br/>/api/v1/data]
    end
    
    subgraph ControllerLayer["Controller Layer"]
        direction TB
        BaseCtrl[BaseController]
        DataCtrl[DataController]
        ProcessCtrl[ProcessController]
        ProjectCtrl[ProjectController]
    end
    
    subgraph ModelLayer["Model Layer"]
        direction TB
        BaseModel[BaseDataModel]
        ProjectModel[ProjectModel]
        AssetModel[AssetModel]
        ChunkModel[ChunkModel]
    end
    
    subgraph StorageLayer["Data Storage"]
        direction LR
        MongoDB[(MongoDB<br/>Database)]
        FileSystem[File System<br/>assets/files/]
    end
    
    subgraph ExternalLayer["External Services"]
        direction TB
        LangChain[LangChain<br/>Document Loaders & Splitters]
    end
    
    subgraph ConfigLayer["Configuration"]
        direction TB
        Config[Settings & Config]
    end
    
    Client -->|HTTP Requests| Router
    Router --> BaseRouter
    Router --> DataRouter
    
    BaseRouter -.->|Welcome| Config
    DataRouter -->|Upload| DataCtrl
    DataRouter -->|Process| ProcessCtrl
    
    DataCtrl -.->|inherits| BaseCtrl
    ProcessCtrl -.->|inherits| BaseCtrl
    ProjectCtrl -.->|inherits| BaseCtrl
    
    DataCtrl -->|uses| ProjectCtrl
    ProcessCtrl -->|uses| ProjectCtrl
    
    DataCtrl -->|queries| ProjectModel
    ProcessCtrl -->|queries| ProjectModel
    ProcessCtrl -->|queries| AssetModel
    ProcessCtrl -->|queries| ChunkModel
    ProjectCtrl -->|queries| ProjectModel
    
    ProjectModel -.->|inherits| BaseModel
    AssetModel -.->|inherits| BaseModel
    ChunkModel -.->|inherits| BaseModel
    
    BaseModel -->|connects| MongoDB
    ProjectModel -->|stores| MongoDB
    AssetModel -->|stores| MongoDB
    ChunkModel -->|stores| MongoDB
    
    DataCtrl -->|writes| FileSystem
    ProcessCtrl -->|reads| FileSystem
    ProjectCtrl -->|manages| FileSystem
    
    ProcessCtrl -->|uses| LangChain
    
    BaseCtrl -.->|reads| Config
    BaseModel -.->|reads| Config
    
    style Client fill:#4A90E2,color:#fff,stroke:#2E5C8A,stroke-width:2px
    style Router fill:#6C7B95,color:#fff,stroke:#4A5568,stroke-width:2px
    style BaseRouter fill:#6C7B95,color:#fff,stroke:#4A5568,stroke-width:2px
    style DataRouter fill:#6C7B95,color:#fff,stroke:#4A5568,stroke-width:2px
    style BaseCtrl fill:#95A5A6,color:#fff,stroke:#7F8C8D,stroke-width:2px
    style DataCtrl fill:#3498DB,color:#fff,stroke:#2980B9,stroke-width:2px
    style ProcessCtrl fill:#3498DB,color:#fff,stroke:#2980B9,stroke-width:2px
    style ProjectCtrl fill:#3498DB,color:#fff,stroke:#2980B9,stroke-width:2px
    style BaseModel fill:#95A5A6,color:#fff,stroke:#7F8C8D,stroke-width:2px
    style ProjectModel fill:#27AE60,color:#fff,stroke:#229954,stroke-width:2px
    style AssetModel fill:#27AE60,color:#fff,stroke:#229954,stroke-width:2px
    style ChunkModel fill:#27AE60,color:#fff,stroke:#229954,stroke-width:2px
    style MongoDB fill:#E74C3C,color:#fff,stroke:#C0392B,stroke-width:2px
    style FileSystem fill:#E67E22,color:#fff,stroke:#D35400,stroke-width:2px
    style LangChain fill:#9B59B6,color:#fff,stroke:#8E44AD,stroke-width:2px
    style Config fill:#34495E,color:#fff,stroke:#2C3E50,stroke-width:2px
    style ClientLayer fill:#ECF0F1,stroke:#BDC3C7,stroke-width:1px
    style APILayer fill:#ECF0F1,stroke:#BDC3C7,stroke-width:1px
    style ControllerLayer fill:#ECF0F1,stroke:#BDC3C7,stroke-width:1px
    style ModelLayer fill:#ECF0F1,stroke:#BDC3C7,stroke-width:1px
    style StorageLayer fill:#ECF0F1,stroke:#BDC3C7,stroke-width:1px
    style ExternalLayer fill:#ECF0F1,stroke:#BDC3C7,stroke-width:1px
    style ConfigLayer fill:#ECF0F1,stroke:#BDC3C7,stroke-width:1px
    
    linkStyle 0 stroke:#00FFFF,stroke-width:3px
    linkStyle 1 stroke:#00FFFF,stroke-width:3px
    linkStyle 2 stroke:#00FFFF,stroke-width:3px
    linkStyle 3 stroke:#00FFFF,stroke-width:3px
    linkStyle 4 stroke:#00FFFF,stroke-width:3px
    linkStyle 5 stroke:#00FFFF,stroke-width:3px
    linkStyle 6 stroke:#00FFFF,stroke-width:3px
    linkStyle 7 stroke:#00FFFF,stroke-width:3px
    linkStyle 8 stroke:#00FFFF,stroke-width:3px
    linkStyle 9 stroke:#00FFFF,stroke-width:3px
    linkStyle 10 stroke:#00FFFF,stroke-width:3px
    linkStyle 11 stroke:#00FFFF,stroke-width:3px
    linkStyle 12 stroke:#00FFFF,stroke-width:3px
    linkStyle 13 stroke:#00FFFF,stroke-width:3px
    linkStyle 14 stroke:#00FFFF,stroke-width:3px
    linkStyle 15 stroke:#00FFFF,stroke-width:3px
    linkStyle 16 stroke:#00FFFF,stroke-width:3px
    linkStyle 17 stroke:#00FFFF,stroke-width:3px
    linkStyle 18 stroke:#00FFFF,stroke-width:3px
    linkStyle 19 stroke:#00FFFF,stroke-width:3px
    linkStyle 20 stroke:#00FFFF,stroke-width:3px
    linkStyle 21 stroke:#00FFFF,stroke-width:3px
    linkStyle 22 stroke:#00FFFF,stroke-width:3px
    linkStyle 23 stroke:#00FFFF,stroke-width:3px
    linkStyle 24 stroke:#00FFFF,stroke-width:3px
    linkStyle 25 stroke:#00FFFF,stroke-width:3px
    linkStyle 26 stroke:#00FFFF,stroke-width:3px
    linkStyle 27 stroke:#00FFFF,stroke-width:3px
    linkStyle 28 stroke:#00FFFF,stroke-width:3px
```

### Architecture Components

#### **API Layer (Routes)**
- **Base Router** (`/api/v1/`): Handles welcome/health check endpoints
- **Data Router** (`/api/v1/data`): Handles data upload and processing endpoints

#### **Controller Layer**
- **BaseController**: Provides common functionality (file paths, random string generation)
- **DataController**: Handles file validation, file path generation, and file name sanitization
- **ProcessController**: Manages document loading (PDF/TXT) and text chunking using LangChain
- **ProjectController**: Manages project directory structure and file organization

#### **Model Layer**
- **BaseDataModel**: Base class for all database models, provides MongoDB connection
- **ProjectModel**: Manages project entities in MongoDB
- **AssetModel**: Manages file assets metadata in MongoDB
- **ChunkModel**: Manages text chunks stored in MongoDB

#### **Data Storage**
- **MongoDB**: Stores projects, assets, and chunks metadata
- **File System**: Stores uploaded files in `assets/files/{project_id}/` directory structure

#### **External Services**
- **LangChain**: Provides document loaders (PDF, TXT) and text splitters for chunking

---

## API Flow Diagrams

### 1. Welcome Endpoint

**Endpoint:** `GET /api/v1/`

This endpoint provides basic application information (name and version).

```mermaid
sequenceDiagram
    participant Client
    participant Router as Base Router
    participant Config as Settings/Config
    participant Response

    Client->>Router: GET /api/v1/
    Router->>Config: Get APP_NAME
    Router->>Config: Get APP_VERSION
    Config-->>Router: Return settings
    Router->>Response: Build response JSON
    Response-->>Client: {app_name, app_version}
```

**Flow Description:**
1. Client sends GET request to `/api/v1/`
2. Router retrieves application name and version from configuration
3. Router returns JSON response with application metadata

---

### 2. Upload Data Endpoint

**Endpoint:** `POST /api/v1/data/upload/{project_id}`

This endpoint handles file uploads, validates files, stores them on the filesystem, and records metadata in the database.

```mermaid
sequenceDiagram
    participant Client
    participant Router as Data Router
    participant ProjectModel
    participant DataCtrl as DataController
    participant ProjectCtrl as ProjectController
    participant FileSystem
    participant AssetModel
    participant MongoDB

    Client->>Router: POST /api/v1/data/upload/{project_id}<br/>+ file (UploadFile)
    
    Router->>ProjectModel: create_instance(db_client)
    ProjectModel->>MongoDB: Initialize collection & indexes
    MongoDB-->>ProjectModel: Collection ready
    
    Router->>ProjectModel: get_project_or_create_one(project_id)
    ProjectModel->>MongoDB: Find project by project_id
    alt Project exists
        MongoDB-->>ProjectModel: Return existing project
    else Project not found
        ProjectModel->>MongoDB: Insert new project
        MongoDB-->>ProjectModel: Return new project
    end
    ProjectModel-->>Router: Project object
    
    Router->>DataCtrl: validate_uploaded_file(file)
    DataCtrl->>DataCtrl: Check file type
    DataCtrl->>DataCtrl: Check file size
    alt File invalid
        DataCtrl-->>Router: (False, error_signal)
        Router-->>Client: 400 Bad Request + error
    else File valid
        DataCtrl-->>Router: (True, success_signal)
        
        Router->>ProjectCtrl: get_project_path(project_id)
        ProjectCtrl->>FileSystem: Create/verify project directory
        FileSystem-->>ProjectCtrl: Project directory path
        ProjectCtrl-->>Router: Project directory path
        
        Router->>DataCtrl: generate_unique_filepath(filename, project_id)
        DataCtrl->>DataCtrl: Clean filename
        DataCtrl->>DataCtrl: Generate unique file ID
        DataCtrl-->>Router: (file_path, file_id)
        
        Router->>FileSystem: Write file content
        alt Write fails
            FileSystem-->>Router: Exception
            Router-->>Client: 400 Bad Request + upload failed
        else Write succeeds
            FileSystem-->>Router: File saved
            
            Router->>AssetModel: create_instance(db_client)
            AssetModel->>MongoDB: Initialize collection & indexes
            MongoDB-->>AssetModel: Collection ready
            
            Router->>AssetModel: create_asset(asset_data)
            AssetModel->>MongoDB: Insert asset document
            MongoDB-->>AssetModel: Return inserted ID
            AssetModel-->>Router: Asset record with ID
            
            Router->>Response: Build success response
            Response-->>Client: 200 OK + {signal, file_id}
        end
    end
```

**Flow Description:**
1. **Project Management**: Router creates/retrieves project from database
2. **File Validation**: DataController validates file type and size
3. **Path Generation**: ProjectController ensures project directory exists, DataController generates unique file path
4. **File Storage**: File is written to filesystem in project-specific directory
5. **Metadata Storage**: AssetModel creates asset record in MongoDB with file metadata
6. **Response**: Returns success signal and file ID, or error if any step fails

**Key Components:**
- **ProjectModel**: Manages project lifecycle (create if not exists)
- **DataController**: Validates files and generates unique file paths
- **ProjectController**: Manages project directory structure
- **AssetModel**: Stores file metadata in database

---

### 3. Process Data Endpoint

**Endpoint:** `POST /api/v1/data/process/{project_id}`

This endpoint processes uploaded files by loading them, splitting into chunks, and storing chunks in the database for RAG operations.

```mermaid
sequenceDiagram
    participant Client
    participant Router as Data Router
    participant ProjectModel
    participant AssetModel
    participant ProcessCtrl as ProcessController
    participant LangChain
    participant ChunkModel
    participant MongoDB
    participant FileSystem

    Client->>Router: POST /api/v1/data/process/{project_id}<br/>+ {chunk_size, overlap_size,<br/>do_reset, file_id?}
    
    Router->>ProjectModel: create_instance(db_client)
    ProjectModel->>MongoDB: Initialize collection
    MongoDB-->>ProjectModel: Collection ready
    
    Router->>ProjectModel: get_project_or_create_one(project_id)
    ProjectModel->>MongoDB: Find or create project
    MongoDB-->>Router: Project object
    
    Router->>AssetModel: create_instance(db_client)
    AssetModel->>MongoDB: Initialize collection
    MongoDB-->>AssetModel: Collection ready
    
    alt file_id provided
        Router->>AssetModel: get_asset_record(project_id, file_id)
        AssetModel->>MongoDB: Find asset by name
        alt Asset not found
            MongoDB-->>AssetModel: None
            AssetModel-->>Router: None
            Router-->>Client: 400 Bad Request + FILE_ID_ERROR
        else Asset found
            MongoDB-->>Router: Asset record
            Note over Router: project_files_ids = {asset_id: file_id}
        end
    else file_id not provided
        Router->>AssetModel: get_all_project_assets(project_id, FILE)
        AssetModel->>MongoDB: Find all project files
        MongoDB-->>AssetModel: List of assets
        AssetModel-->>Router: All project files
        Note over Router: project_files_ids = {all assets}
    end
    
    alt No files found
        Router-->>Client: 400 Bad Request + NO_FILES_ERROR
    else Files found
        Router->>ChunkModel: create_instance(db_client)
        ChunkModel->>MongoDB: Initialize collection
        MongoDB-->>ChunkModel: Collection ready
        
        alt do_reset == 1
            Router->>ChunkModel: delete_chunks_by_project_id(project_id)
            ChunkModel->>MongoDB: Delete all project chunks
            MongoDB-->>ChunkModel: Deletion count
        end
        
        Router->>ProcessCtrl: ProcessController(project_id)
        
        loop For each file in project_files_ids
            Router->>ProcessCtrl: get_file_content(file_id)
            ProcessCtrl->>ProcessCtrl: get_file_extension(file_id)
            ProcessCtrl->>FileSystem: Read file from project directory
            alt File not found
                FileSystem-->>ProcessCtrl: None
                ProcessCtrl-->>Router: None
                Note over Router: Log error, continue to next file
            else File found
                FileSystem-->>ProcessCtrl: File path
                ProcessCtrl->>LangChain: Load document (TextLoader/PyMuPDFLoader)
                LangChain-->>ProcessCtrl: Document objects
                ProcessCtrl-->>Router: File content (list of documents)
                
                Router->>ProcessCtrl: process_file_content(content, chunk_size, overlap_size)
                ProcessCtrl->>LangChain: RecursiveCharacterTextSplitter
                LangChain->>LangChain: Split documents into chunks
                LangChain-->>ProcessCtrl: List of chunks
                ProcessCtrl-->>Router: File chunks
                
                alt Chunks empty
                    Router-->>Client: 400 Bad Request + PROCESSING_FAILED
                else Chunks created
                    Router->>Router: Create DataChunk objects
                    Router->>ChunkModel: insert_many_chunks(chunks)
                    ChunkModel->>MongoDB: Bulk insert chunks (batched)
                    MongoDB-->>ChunkModel: Insert count
                    ChunkModel-->>Router: Number of inserted chunks
                    Note over Router: no_records += inserted_count<br/>no_files += 1
                end
            end
        end
        
        Router->>Response: Build success response
        Response-->>Client: 200 OK + {signal, inserted_chunks, processed_files}
    end
```

**Flow Description:**
1. **Project Validation**: Router retrieves or creates project
2. **File Selection**: 
   - If `file_id` provided: Fetch specific file asset
   - If not provided: Fetch all project files
3. **Reset Option**: If `do_reset=1`, delete existing chunks for the project
4. **File Processing Loop**:
   - Load file content using LangChain loaders (PDF or TXT)
   - Split content into chunks using RecursiveCharacterTextSplitter
   - Create DataChunk objects with metadata
   - Bulk insert chunks into MongoDB
5. **Response**: Returns success signal with counts of inserted chunks and processed files

**Key Components:**
- **ProcessController**: Handles document loading and chunking logic
- **LangChain**: Provides document loaders (TextLoader, PyMuPDFLoader) and text splitters
- **ChunkModel**: Manages chunk storage with batch insertion
- **AssetModel**: Retrieves file metadata for processing

**Processing Parameters:**
- `chunk_size`: Size of each text chunk (default: 100)
- `overlap_size`: Overlap between chunks (default: 20)
- `do_reset`: Whether to delete existing chunks before processing (0 or 1)
- `file_id`: Optional specific file to process, or process all files if omitted

---

## Important Considerations

1. **Database Connection**: MongoDB connection is established at application startup and stored in `app.db_client` for use across all requests.

2. **File Storage**: Files are stored in a local filesystem structure under `src/assets/files/{project_id}/` with unique file IDs to prevent naming conflicts.

3. **Supported File Types**: The system supports PDF and TXT files based on the ProcessController implementation. File type validation is handled in DataController using configuration settings.

4. **Error Handling**: All endpoints return JSON responses with appropriate HTTP status codes. Error responses include a `signal` field indicating the error type.

5. **Chunking Strategy**: Uses LangChain's RecursiveCharacterTextSplitter with configurable chunk size and overlap. Chunks are stored with metadata including order, project ID, and asset ID.

6. **Project Isolation**: Each project has its own directory and database records are scoped by project ID, ensuring data isolation between projects.

7. **Batch Processing**: Chunk insertion uses batch operations (default batch size: 100) for efficient database writes.

8. **Configuration**: Application settings are loaded from environment variables via Pydantic Settings, including MongoDB connection, file size limits, and allowed file types.

---

## Additional Notes

- The system uses **async/await** patterns throughout for non-blocking I/O operations.
- All database models follow a factory pattern with `create_instance()` class methods for proper initialization.
- Controllers inherit from `BaseController` which provides common utilities and configuration access.
- Models inherit from `BaseDataModel` which provides database client access and settings.



