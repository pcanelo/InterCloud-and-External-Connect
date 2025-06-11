
# Comparación: AK/SK vs IAM Roles Anywhere

## 1. Diferencia entre AK/SK y IAM Roles Anywhere

| Aspecto                        | AK/SK (AccessKey/SecretKey)                         | IAM Roles Anywhere                                 |
|-------------------------------|-----------------------------------------------------|---------------------------------------------------|
| Tipo de credencial            | Estática (permanente, tipo `AKIA...`)               | Temporal (STS), autenticación por certificado     |
| Forma de autenticación        | Firma HMAC con SecretKey                            | Firma con certificado X.509                       |
| Gestión                       | Manual: debes entregar y rotar el AK/SK             | Automática: se usan certificados firmados         |
| Persistencia                  | Vive hasta que lo borres o rotees manualmente       | Expira según duración STS (ej. 15-60 min)         |
| Seguridad                     | Bajo si el AK/SK se filtra                          | Alto: tokens temporales, sin clave secreta        |
| Escalabilidad                 | Difícil (más claves = más riesgo)                   | Mejor, usa certificados con control centralizado  |
| Auditable                     | Parcial (requiere buen tagging)                     | Total: cada `AssumeRole` queda en CloudTrail      |
| Restricción por host          | No se puede controlar desde qué host se usa         | Puede vincularse a identidad de máquina (cert)    |

## 2. Seguridad reforzada con IAM Roles Anywhere

| Refuerzo de Seguridad                      | Explicación                                                                 |
|-------------------------------------------|-----------------------------------------------------------------------------|
| Sin claves permanentes expuestas          | Elimina el riesgo de que alguien robe `AKIA...`/`SECRET...`                |
| Credenciales STS temporales               | Aunque roben una sesión, dura 15–60 min máximo                             |
| Certificados X.509 firmados               | La identidad se basa en criptografía, no en texto plano                    |
| Vinculación a CA confiable                | Solo certificados emitidos por tu CA pueden autenticarse                   |
| Auditoría completa en CloudTrail          | Cada `AssumeRole` queda registrado con fuente y parámetros                 |
| Scoped-down Role Policies                 | Puedes definir permisos mínimos por rol, incluso por atributo              |
| Revocación controlada                     | Puedes revocar el certificado o la CA sin cambiar usuarios ni claves       |
