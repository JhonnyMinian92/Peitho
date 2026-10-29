# ADR-0002: autenticación local segura en el despliegue inicial

- **Estado:** Aceptado
- **Fecha:** 2026-10-29
- **Ámbito:** login, usuarios, sesiones y autorización

## Contexto
El despliegue inicial debe evitar costes y dependencias de proveedores externos de identidad sin reducir los controles de seguridad para un producto multiempresa.

## Decisión
Peitho comienza con **autenticación local segura**, gestión propia de usuarios y roles por empresa y sin federación inicial con Microsoft Entra, Google Workspace, Keycloak u otro proveedor externo. OIDC/SAML quedan como extensiones futuras sujetas a decisión documentada y necesidad de negocio verificable.

## Controles requeridos
- Contraseñas con hash Argon2id; nunca cifrado reversible de contraseñas.
- MFA mediante TOTP o passkeys/WebAuthn; MFA obligatorio para administradores y operaciones de riesgo según política.
- TLS/HTTPS, sesiones/cookies seguras, protección CSRF según topología, rate limiting, recuperación controlada, revocación de sesiones y auditoría.
- Cifrado de aplicación para intercambio de credenciales cuando corresponda como capa adicional; no reemplaza TLS, MFA ni hashing.
- Aislamiento tenant/roles comprobado en backend para endpoints, exportaciones, tareas, herramientas MCP y APIs B2B.
- No registrar contraseñas, códigos/semillas MFA, tokens, claves privadas ni ciphertext sensible.
- Diseñar una interfaz extensible para añadir OIDC/SAML sin acoplar la lógica de dominio a un proveedor.

## Consecuencias
Se reduce la dependencia inicial de terceros; Peitho asume el ciclo de vida de cuentas, recuperación, auditoría y operación de credenciales.

## Criterios de aceptación
- Pruebas de login, intentos abusivos, replay, expiración, MFA, recuperación, revocación y sesiones.
- Pruebas de aislamiento entre empresas y de permisos backend, no solo navegación Angular.
- Sin secretos en configuración, repositorio o logs.
