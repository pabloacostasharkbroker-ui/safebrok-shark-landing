# Acceso progresivo con Google — SafeBrok Landing

Estado: **arquitectura preparada; autenticación NO activada**.

## Aislamiento obligatorio
- Repositorio exclusivo: `pabloacostasharkbroker-ui/safebrok-shark-landing`.
- Crear un **nuevo proyecto Supabase independiente**, destinado solo a prospectos SafeBrok.
- No usar el proyecto Supabase del HUB, sus tablas `profiles`, `holds`, `developments`, ni sus roles, políticas RLS o credenciales.
- No modificar el repositorio del HUB.

## Experiencia de usuario
1. Landing, portafolio, imágenes y contenido introductorio: **públicos**.
2. Cuando se solicite un recurso detallado o asesoría: ofrecer `Continuar con Google` y una explicación del uso de datos.
3. Autenticar por OAuth de Google mediante Supabase Auth y retornar a la misma landing.
4. Mostrar la información de sesión y un botón de cerrar sesión; no bloquear la navegación pública.
5. Solicitar WhatsApp únicamente como campo opcional y con consentimiento específico.

## Diseño de datos (propuesto, no ejecutado)
- `prospects`: `user_id uuid primary key references auth.users(id)`, `display_name text`, `email text`, `created_at timestamptz`, `last_login_at timestamptz`.
- `prospect_events`: `id uuid primary key`, `user_id uuid references auth.users(id)`, `event_type text`, `project_slug text`, `created_at timestamptz`.
- Eventos mínimos: `sign_in`, `project_view`, `consultation_request`. Evitar capturar búsquedas privadas, coordenadas, IP o huellas digitales.
- Habilitar RLS en ambas tablas. Los prospectos pueden consultar únicamente sus propios datos. La consulta agregada de administración se hará mediante backend autenticado con permisos separados, nunca mediante una clave privilegiada en el navegador.
- Política de retención, eliminación de datos y consentimiento documentados antes de iniciar la captura.

## Integración técnica pendiente
1. Crear proyecto Supabase SafeBrok exclusivo.
2. Crear credenciales OAuth en Google Cloud; habilitar proveedor Google en Supabase Auth.
3. Configurar URL de sitio `https://safebrok-shark-landing.pages.dev` y redirecciones permitidas para producción y pruebas.
4. Publicar aviso de privacidad completo y consentimiento correspondiente antes de registrar actividad vinculada a identidad.
5. Implementar cliente Supabase con URL de proyecto y clave **publicable** únicamente; jamás `service_role` o clave secreta en frontend.
6. Implementar `signInWithOAuth({provider:'google',options:{redirectTo:...}})`, recuperación de sesión, cierre de sesión y registro mínimo de eventos.
7. Probar acceso, retorno, cierre de sesión, rechazo de acceso a datos de terceros y experiencia móvil.
8. Activar en producción solo tras superar las pruebas.

## Criterios de seguridad y medición
- Sin login, no atribuir identidad a visitas anónimas.
- Google no proporciona automáticamente teléfono ni WhatsApp.
- No guardar tokens OAuth, datos sensibles ni identificadores de terceros en logs públicos.
- Las imágenes y archivos alojados como recursos estáticos públicos **no son privados** por ocultarlos tras un botón. Los recursos realmente restringidos deben servirse desde almacenamiento privado con autorización comprobada en el servidor.
- No confundir autenticación (quién es) con autorización (qué puede ver).
- La landing actual permanece funcional durante la preparación.

## Referencias
- https://supabase.com/docs/guides/auth/social-login/auth-google
- https://supabase.com/docs/guides/auth/redirect-urls
- https://supabase.com/docs/guides/database/postgres/row-level-security
