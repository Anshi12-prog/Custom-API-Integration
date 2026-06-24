```mermaid
flowchart LR
    CFO[Finance Leadership / CFO]
    AUDIT[Internal Audit]
    OPS[Operations & Cost Controllers]
    SAP[SAP S/4HANA 2023 FPS02]
    INT[Integration Platform]
    FIN[Zetheta FinSight 4.2]
    SNOW[Snowflake]
    AAD[Azure AD]
    MON[Monitoring Stack<br/>Nagios + Grafana]

    CFO --> FIN
    AUDIT --> FIN
    OPS --> FIN

    SAP --> INT
    INT --> FIN
    INT --> SNOW
    AAD --> INT
    AAD --> FIN
    INT --> MON
