# Questions Pending

## Q-INV-01 (medium, architecture, opcional)
¿Cuál es el mecanismo preferido para la integración de lectura con Inventary en el entorno de producción: réplica lógica de PostgreSQL dedicada, lectura directa sobre réplica transaccional de lectura (read-replica), o extracción vía API interna de solo lectura?

Why: Determina si el adaptador de ingesta se configurará para conectarse directamente mediante SQLAlchemy a una base de lectura réplica o si consumirá endpoints internos de CDC/Outbox provistos por el equipo de Inventary.

Answer: 

## Q-AUTH-01 (medium, security, opcional)
¿Se requerirá federación con un proveedor de identidad corporativo específico (como Azure AD / Microsoft Entra, Google Workspace o Keycloak) para el inicio de sesión corporativo desde el despliegue inicial, o se comenzará con la autenticación local segura con cifrado de aplicación y MFA?

Why: Permite priorizar los conectores OIDC/SAML en la configuración del servicio de autenticación desde el Módulo 2 o dejarlos preparados como interfaz extensible.

Answer: 

## Q-LLM-01 (low, architecture, opcional)
¿Qué proveedor de modelos de lenguaje e incrustaciones (embeddings) se configurará inicialmente en la pasarela de IA de Peitho (por ejemplo: OpenAI API, Azure OpenAI, Anthropic Claude, Google Vertex AI, o una instancia local con Ollama/vLLM)?

Why: Permite preparar las credenciales y bibliotecas SDK correspondientes en el AI Policy Gateway y calibrar las dimensiones vectoriales de pgvector (ej. 1536 o 3072 dimensiones).

Answer:
