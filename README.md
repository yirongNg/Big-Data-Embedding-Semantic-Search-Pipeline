# Big-Data-Embedding-Semantic-Search-Pipeline

```mermaid
graph TD
    A[AWS S3 Bucket: Raw Text Data] -->|Distributed Ingestion| B[PySpark DataFrame]
    B -->|Data Cleansing & Partitioning| C[Optimized PySpark Tasks]
    C -->|Rate-Limited Batch UDF| D[OpenAI API: text-embedding-3-small]
    D -->|1536-Dimensional Vectors| E[PySpark foreachPartition]
    E -->|Bulk Batch Upsert| F[Pinecone Serverless Vector DB]
    
    G[User Search Query] -->|Single Embed Request| H[OpenAI API]
    H -->|Query Vector| I[Pinecone Index Query]
    F -->|Top K Semantic Matches| I
    I -->|Instant Contextual Results| J[Streamlit User Interface]
    
    style B fill:#f9f,stroke:#333,stroke-width:2px
    style F fill:#bbf,stroke:#333,stroke-width:2px
```
```mermaid
 graph TD
    subgraph "Data Lake (Storage Layer)"
        S3[AWS S3 Bucket]
        Raw[Raw CSV/Parquet Files]
        S3 --> Raw
    end

    subgraph "Processing Cluster (Compute Layer)"
        Spark[Apache Spark Driver]
        Worker1[Worker Node 1]
        Worker2[Worker Node 2]
        Worker3[Worker Node 3]
        
        Raw -->|Read Distributively| Spark
        Spark -->|Partition Data| Worker1
        Spark -->|Partition Data| Worker2
        Spark -->|Partition Data| Worker3
    end

    subgraph "Intelligence Layer (API Integration)"
        OpenAI[OpenAI API (Embeddings)]
        
        Worker1 -.->|Batch Request w/ Backoff| OpenAI
        Worker2 -.->|Batch Request w/ Backoff| OpenAI
        Worker3 -.->|Batch Request w/ Backoff| OpenAI
        
        OpenAI -.->|Return Vectors (1536d)| Worker1
        OpenAI -.->|Return Vectors (1536d)| Worker2
        OpenAI -.->|Return Vectors (1536d)| Worker3
    end

    subgraph "Serving Layer (Vector Database)"
        Pinecone[Pinecone Serverless Index]
        
        Worker1 -->|Batch Upsert| Pinecone
        Worker2 -->|Batch Upsert| Pinecone
        Worker3 -->|Batch Upsert| Pinecone
    end

    subgraph "User Application"
        UI[Streamlit Dashboard]
        User[End User]
        
        User -->|Query| UI
        UI -->|Generate Query Vector| OpenAI
        UI -->|Semantic Search| Pinecone
        Pinecone -->|Top-K Results| UI
    end

    style S3 fill:#E1F5FE,stroke:#01579B,stroke-width:2px
    style Spark fill:#FFF3E0,stroke:#E65100,stroke-width:2px
    style OpenAI fill:#F3E5F5,stroke:#4A148C,stroke-width:2px
    style Pinecone fill:#E8F5E9,stroke:#1B5E20,stroke-width:2px
```

