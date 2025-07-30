 # DER Ancora Play v4

```mermaid
erDiagram
    users {
        id BIGINT PK "GENERATED ALWAYS AS IDENTITY"
        email VARCHAR UK "UNIQUE NOT NULL"
        display_name VARCHAR "NOT NULL"
        status common_status "DEFAULT 'ACTIVE'"
        created_at TIMESTAMPTZ "DEFAULT now()"
        updated_at TIMESTAMPTZ "DEFAULT now()"
    }

    roles {
        id BIGINT PK "GENERATED ALWAYS AS IDENTITY"
        code VARCHAR UK "UNIQUE NOT NULL"
        description TEXT "NOT NULL"
    }

    user_roles {
        user_id BIGINT PK,FK
        role_id BIGINT PK,FK
    }

    user_groups {
        id BIGINT PK "GENERATED ALWAYS AS IDENTITY"
        code VARCHAR UK "UNIQUE NOT NULL"
        name VARCHAR "NOT NULL"
        description TEXT
        status common_status "DEFAULT 'ACTIVE'"
        created_at TIMESTAMPTZ "DEFAULT now()"
        updated_at TIMESTAMPTZ "DEFAULT now()"
    }

    channels {
        id BIGINT PK "GENERATED ALWAYS AS IDENTITY"
        name VARCHAR UK "UNIQUE NOT NULL"
        slug VARCHAR UK "UNIQUE NOT NULL"
        description TEXT
        visibility visibility_type "DEFAULT 'PRIVATE'"
        status common_status "DEFAULT 'ACTIVE'"
        created_by BIGINT FK
        created_at TIMESTAMPTZ "DEFAULT now()"
        updated_at TIMESTAMPTZ "DEFAULT now()"
    }

    channel_user_groups {
        channel_id BIGINT PK,FK
        user_group_id BIGINT PK,FK
    }

    folders {
        id BIGINT PK "GENERATED ALWAYS AS IDENTITY"
        channel_id BIGINT FK
        name VARCHAR "NOT NULL"
        description TEXT
        is_default BOOLEAN "DEFAULT false"
        status common_status "DEFAULT 'ACTIVE'"
        created_by BIGINT FK
        created_at TIMESTAMPTZ "DEFAULT now()"
        updated_at TIMESTAMPTZ "DEFAULT now()"
    }

    media_types {
        id BIGINT PK "GENERATED ALWAYS AS IDENTITY"
        code VARCHAR UK "UNIQUE NOT NULL"
        name VARCHAR "NOT NULL"
        mime_types TEXT[] "Array de MIME types"
        extensions TEXT[] "Array de extensões"
        embed_template TEXT
        description TEXT
    }

    providers {
        id BIGINT PK "GENERATED ALWAYS AS IDENTITY"
        code VARCHAR UK "UNIQUE NOT NULL"
        name VARCHAR "NOT NULL"
        base_url VARCHAR
        url_patterns JSONB
        api_config JSONB
        validation_rules JSONB
        embed_template TEXT
        status common_status "DEFAULT 'ACTIVE'"
        description TEXT
    }

    contents {
        id BIGINT PK "GENERATED ALWAYS AS IDENTITY"
        external_id UUID UK "DEFAULT gen_random_uuid()"
        folder_id BIGINT FK
        media_type_id BIGINT FK "NOT NULL"
        provider_id BIGINT FK "NOT NULL"
        title VARCHAR "NOT NULL"
        description TEXT
        url VARCHAR "NOT NULL"
        status common_status "DEFAULT 'ACTIVE'"
        visibility visibility_type "DEFAULT 'PRIVATE'"
        only_in_trail BOOLEAN "DEFAULT false"
        metadata JSONB
        created_by BIGINT FK
        created_at TIMESTAMPTZ "DEFAULT now()"
        updated_at TIMESTAMPTZ "DEFAULT now()"
    }

    content_search {
        content_id BIGINT PK,FK
        search_vec TSVECTOR "NOT NULL"
    }

    tags {
        id BIGINT PK "GENERATED ALWAYS AS IDENTITY"
        name VARCHAR UK "UNIQUE NOT NULL"
        created_at TIMESTAMPTZ "DEFAULT now()"
    }

    content_tags {
        content_id BIGINT PK,FK
        tag_id BIGINT PK,FK
    }

    trails {
        id BIGINT PK "GENERATED ALWAYS AS IDENTITY"
        folder_id BIGINT FK
        name VARCHAR "NOT NULL"
        description TEXT
        status common_status "DEFAULT 'ACTIVE'"
        visibility visibility_type "DEFAULT 'PRIVATE'"
        created_by BIGINT FK
        created_at TIMESTAMPTZ "DEFAULT now()"
        updated_at TIMESTAMPTZ "DEFAULT now()"
    }

    trail_contents {
        trail_id BIGINT PK,FK
        content_id BIGINT PK,FK
        display_order INTEGER "NOT NULL"
    }

    user_content_progress {
        user_id BIGINT PK,FK
        content_id BIGINT PK,FK
        first_viewed_at TIMESTAMPTZ "NOT NULL"
        last_viewed_at TIMESTAMPTZ "NOT NULL"
        progress_seconds INTEGER "DEFAULT 0"
        total_duration INTEGER
        view_count INTEGER "DEFAULT 1"
        completed BOOLEAN "DEFAULT false"
        completed_at TIMESTAMPTZ
    }

    %% Relacionamentos Principais
    users ||--o{ user_roles : "has"
    roles ||--o{ user_roles : "assigned to"
    
    users ||--o{ channels : "creates"
    channels ||--o{ channel_user_groups : "has access"
    user_groups ||--o{ channel_user_groups : "can access"
    
    channels ||--o{ folders : "contains"
    users ||--o{ folders : "creates"
    
    media_types ||--o{ contents : "categorizes"
    providers ||--o{ contents : "hosts"
    folders ||--o{ contents : "contains"
    users ||--o{ contents : "creates"
    
    contents ||--|| content_search : "indexed by"
    
    contents ||--o{ content_tags : "tagged with"
    tags ||--o{ content_tags : "tags"
    
    folders ||--o{ trails : "organizes"
    users ||--o{ trails : "creates"
    trails ||--o{ trail_contents : "contains"
    contents ||--o{ trail_contents : "part of"
    
    users ||--o{ user_content_progress : "tracks"
    contents ||--o{ user_content_progress : "viewed by"
