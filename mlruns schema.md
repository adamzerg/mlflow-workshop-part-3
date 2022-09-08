
sqlite> .database
main: D:\OneDrive\GitHub\mlflow-workshop-part-3\src\mlruns.db r/w

sqlite> .tables
alembic_version        metrics                registered_model_tags
experiment_tags        model_version_tags     registered_models
experiments            model_versions         runs
latest_metrics         params                 tags

sqlite> .schema alembic_version
CREATE TABLE alembic_version (
        version_num VARCHAR(32) NOT NULL,
        CONSTRAINT alembic_version_pkc PRIMARY KEY (version_num)
);

sqlite> .schema registered_models
CREATE TABLE registered_models (
        name VARCHAR(256) NOT NULL,
        creation_time BIGINT,
        last_updated_time BIGINT,
        description VARCHAR(5000),
        CONSTRAINT registered_model_pk PRIMARY KEY (name),
        UNIQUE (name)
);

sqlite> .schema registered_model_tags  
CREATE TABLE registered_model_tags (
        "key" VARCHAR(250) NOT NULL,
        value VARCHAR(5000),
        name VARCHAR(256) NOT NULL,
        CONSTRAINT registered_model_tag_pk PRIMARY KEY ("key", name),
        FOREIGN KEY(name) REFERENCES registered_models (name) ON UPDATE cascade
);

sqlite> .schema model_versions
CREATE TABLE IF NOT EXISTS "model_versions" (
        name VARCHAR(256) NOT NULL,
        version INTEGER NOT NULL,
        creation_time BIGINT,
        last_updated_time BIGINT,
        description VARCHAR(5000),
        user_id VARCHAR(256),
        current_stage VARCHAR(20),
        source VARCHAR(500),
        run_id VARCHAR(32),
        status VARCHAR(20),
        status_message VARCHAR(500),
        run_link VARCHAR(500),
        CONSTRAINT model_version_pk PRIMARY KEY (name, version),
        FOREIGN KEY(name) REFERENCES registered_models (name) ON UPDATE CASCADE
);

sqlite> .schema model_version_tags
CREATE TABLE model_version_tags (
        "key" VARCHAR(250) NOT NULL,
        value VARCHAR(5000),
        name VARCHAR(256) NOT NULL,
        version INTEGER NOT NULL,
        CONSTRAINT model_version_tag_pk PRIMARY KEY ("key", name, version),
        FOREIGN KEY(name, version) REFERENCES model_versions (name, version) ON UPDATE cascade
);

sqlite> .schema experiments
CREATE TABLE experiments (
        experiment_id INTEGER NOT NULL,
        name VARCHAR(256) NOT NULL,
        artifact_location VARCHAR(256),
        lifecycle_stage VARCHAR(32),
        CONSTRAINT experiment_pk PRIMARY KEY (experiment_id),
        CONSTRAINT experiments_lifecycle_stage CHECK (lifecycle_stage IN ('active', 'deleted')),
        UNIQUE (name)
);

sqlite> .schema experiment_tags  
CREATE TABLE experiment_tags (
        "key" VARCHAR(250) NOT NULL,
        value VARCHAR(5000),
        experiment_id INTEGER NOT NULL,
        CONSTRAINT experiment_tag_pk PRIMARY KEY ("key", experiment_id),
        FOREIGN KEY(experiment_id) REFERENCES experiments (experiment_id)
);

sqlite> .schema runs         
CREATE TABLE IF NOT EXISTS "runs" (
        run_uuid VARCHAR(32) NOT NULL,
        name VARCHAR(250),
        source_type VARCHAR(20),
        source_name VARCHAR(500),
        entry_point_name VARCHAR(50),
        user_id VARCHAR(256),
        status VARCHAR(9),
        start_time BIGINT,
        end_time BIGINT,
        source_version VARCHAR(50),
        lifecycle_stage VARCHAR(20),
        artifact_uri VARCHAR(200),
        experiment_id INTEGER, deleted_time BIGINT,
        CONSTRAINT run_pk PRIMARY KEY (run_uuid),
        CONSTRAINT runs_lifecycle_stage CHECK (lifecycle_stage IN ('active', 'deleted')),
        CONSTRAINT source_type CHECK (source_type IN ('NOTEBOOK', 'JOB', 'LOCAL', 'UNKNOWN', 'PROJECT')),
        CHECK (status IN ('SCHEDULED', 'FAILED', 'FINISHED', 'RUNNING', 'KILLED')),
        FOREIGN KEY(experiment_id) REFERENCES experiments (experiment_id)
);

sqlite> .schema tags          
CREATE TABLE IF NOT EXISTS "tags" (
        "key" VARCHAR(250) NOT NULL,
        value VARCHAR(5000),
        run_uuid VARCHAR(32) NOT NULL,
        CONSTRAINT tag_pk PRIMARY KEY ("key", run_uuid),
        FOREIGN KEY(run_uuid) REFERENCES runs (run_uuid)
);
CREATE INDEX index_tags_run_uuid ON tags (run_uuid);

sqlite> .schema params
CREATE TABLE IF NOT EXISTS "params" (
        "key" VARCHAR(250) NOT NULL,
        value VARCHAR(500) NOT NULL,
        run_uuid VARCHAR(32) NOT NULL,
        CONSTRAINT param_pk PRIMARY KEY ("key", run_uuid),
        FOREIGN KEY(run_uuid) REFERENCES runs (run_uuid)
);
CREATE INDEX index_params_run_uuid ON params (run_uuid);

sqlite> .schema metrics    
CREATE TABLE IF NOT EXISTS "metrics" (
        "key" VARCHAR(250) NOT NULL,
        value FLOAT NOT NULL,
        timestamp BIGINT NOT NULL,
        run_uuid VARCHAR(32) NOT NULL,
        step BIGINT DEFAULT '0' NOT NULL,
        is_nan BOOLEAN DEFAULT '0' NOT NULL,
        CONSTRAINT metric_pk PRIMARY KEY ("key", timestamp, step, run_uuid, value, is_nan), 
        CHECK (is_nan IN (0, 1)),
        FOREIGN KEY(run_uuid) REFERENCES runs (run_uuid)
);
CREATE INDEX index_metrics_run_uuid ON metrics (run_uuid);

sqlite> .schema latest_metrics 
CREATE TABLE IF NOT EXISTS "latest_metrics" (
        "key" VARCHAR(250) NOT NULL,
        value FLOAT NOT NULL,
        timestamp BIGINT,
        step BIGINT NOT NULL,
        is_nan BOOLEAN NOT NULL,
        run_uuid VARCHAR(32) NOT NULL,
        CONSTRAINT latest_metric_pk PRIMARY KEY ("key", run_uuid),
        CHECK (is_nan IN (0, 1)),
        FOREIGN KEY(run_uuid) REFERENCES runs (run_uuid)
);
CREATE INDEX index_latest_metrics_run_uuid ON latest_metrics (run_uuid);

