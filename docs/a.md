graph TD
    subgraph "1. Nguồn dữ liệu (Data Source)"
        S3[(Amazon S3\nRaw CSV Files)]
    end

    subgraph "2. Điều phối & Môi trường chạy (Orchestration - Astro/Docker)"
        Airflow((Apache Airflow))
        Cosmos[Astronomer Cosmos]
        dbt[dbt-core\n(Virtual Env)]
        
        Airflow -->|1. Chạy lệnh Load| SQLExec[SQLExecuteQueryOperator]
        Airflow -->|2. Kích hoạt Transform| Cosmos
        Cosmos -->|Quản lý & Chạy| dbt
    end

    subgraph "3. Nền tảng Dữ liệu Đám mây (Snowflake Data Warehouse)"
        RAW[(RAW Layer\nDatabase: RAW.BNPL)]
        STG[(Staging Layer / Bronze\nSchema: stg)]
        MARTS[(Marts Layer / Gold\nSchema: marts)]

        SQLExec -.->|Thực thi COPY INTO| RAW
        dbt -.->|Biên dịch & Chạy SQL| STG
        dbt -.->|Biên dịch & Chạy SQL| MARTS

        RAW -->|Clean & Standardize| STG
        STG -->|Star Schema / Denormalize| MARTS
    end

    subgraph "4. Tiêu thụ dữ liệu (Consumption)"
        BI[BI Tools\nPower BI / Tableau]
        Stakeholders[Data Analysts\nBusiness Users]
    end

    S3 ==>|COPY INTO| RAW
    MARTS ==> BI
    BI --> Stakeholders

    classDef aws fill:#FF9900,stroke:#232F3E,stroke-width:2px,color:white;
    classDef airflow fill:#017CEE,stroke:#017CEE,stroke-width:2px,color:white;
    classDef snowflake fill:#29B5E8,stroke:#29B5E8,stroke-width:2px,color:white;
    classDef dbt fill:#FF6B4A,stroke:#FF6B4A,stroke-width:2px,color:white;
    
    class S3 aws;
    class Airflow,SQLExec airflow;
    class RAW,STG,MARTS snowflake;
    class dbt,Cosmos dbt;
