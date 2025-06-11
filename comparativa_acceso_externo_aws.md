
# Comparativa de métodos para acceso desde External Cloud to AWS --> Especificamente S3 

## Métodos evaluados

| Método                           | Descripción                                                                 | Seguridad                                      | Costo aproximado                | Comentarios clave                                                                 |
|----------------------------------|-----------------------------------------------------------------------------|------------------------------------------------|----------------------------------|------------------------------------------------------------------------------------|
| AK/SK (Access Key / Secret Key) | Claves estáticas generadas manualmente                                     | Muy baja: exposición directa, difícil rotación | Bajo (pero alto riesgo oculto)  | No recomendado para flujos CI/CD externos. Claves pueden filtrarse fácilmente.     |
| Identity Federation (OIDC)      | Federación vía Entra External ID + STS                                     | Alta: tokens firmados, control por claims      | Bajo a medio                     | Ideal para terceros. Basado en estándares seguros (JWT + STS).                     |
| IAM Roles Anywhere              | STS + certificados X.509 verificados por CA registrada                      | Alta: sin claves, tokens temporales            | Medio                           | Útil para workloads externos con control sobre certificados.                       |
| VPN + VPC Endpoint Gateway     | Canal cifrado entre redes y acceso vía Endpoint Gateway a S3                | Alta, pero compleja                            | Medio a alto                    | Más apropiado para conexiones continuas entre infraestructuras.                    |
| Direct Connect + VGW + Endpoint| Enlace dedicado físico + acceso directo a S3                                | Muy alta, pero caro                            | Alto                            | Muy seguro, pero sobreprovisionado si solo se necesita subir bundles esporádicos. |

## Recomendación

Para el caso de uso **CI/CD desde Tencent, GCP, Alibaba, Huawei u otros, podran subir bundles a S3 o usar otros recurso**, lo más seguro, escalable y administrable es:

- **Identity Federation (OIDC + Entra External ID)**
- O bien **IAM Roles Anywhere** si se gestiona bien la CA y certificados

**Evitar completamente el uso de AK/SK.**
