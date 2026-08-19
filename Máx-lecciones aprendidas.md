MAX - Lecciones 
Aprendidas 
Proyecto: MAX SaberPROPeríodo: 6 de diciembre 2025 - 5 de enero 2026Rol: Desarrollador / Mantenedor
  
1.	LECCIONES TÉCNICAS 
Arquitectura y Diseño 
Lección 1: Verificar estructura de datos completa antes de implementar 
●	Problema: El panel de progreso se rompió 4-5 veces por asumir la estructura de DynamoDB 
●	Solución: Revisar la estructura completa de datos al inicio 
●	Aplicación: Siempre hacer get_student_status() y revisar el JSON completo antes de escribir código 
Lección 2: Merge profundo vs. merge superficial 
●	Problema: Merge superficial sobrescribía datos existentes en DynamoDB 
● 	Solución: Implementar merge profundo para objetos anidados (overview, 	engagement
) 
●	Aplicación: Para objetos anidados, hacer merge nivel por nivel, no solo {**dict1, **dict2} 
Lección 3: Números como strings desde DynamoDB 
●	Problema: DynamoDB devuelve números como strings, causando errores en comparaciones 
●	Solución: Siempre usar parseInt() o int() al leer números de DynamoDB 
●	Aplicación: Asumir que todo número de DynamoDB viene como string y parsearlo 
Backend (Python) 
Lección 4: Cálculo de logros en backend, no frontend 
●	Problema: Logros se "reseteaban" al recargar porque se calculaban en frontend 
●	Solución: Mover cálculo de logros al backend y persistir en DynamoDB 
●	Aplicación: La fuente de verdad debe estar en el backend/base de datos, no en el cliente 
Lección 5: Preservar datos existentes al actualizar 
● 	Problema: upsert_student_status con 	overwrite=False	 aún podía perder datos
●	Solución: Verificar explícitamente qué campos preservar si no vienen en el nuevo status 
●	Aplicación: Lista explícita de campos críticos a preservar: competencies, achievements 
Frontend (JavaScript) 
Lección 6: Race conditions con setTimeout 
●	Problema: lastStreamingMessage se limpiaba antes de que setTimeout accediera 
● 	Solución: Guardar referencia local del elemento antes del 	setTimeout
● 	Aplicación: Si usas 	setTimeout	 con variables que pueden cambiar, guarda una copia local
Lección 7: Renderizado condicional de datos 
●	Problema: Panel no se mostraba cuando pendingProgressResolve era null 
●	Solución: Siempre renderizar con datos recibidos, independientemente del estado de promesas 
●	Aplicación: No depender de una sola ruta de datos; manejar múltiples escenarios 
Infraestructura (AWS) 
Lección 8: S3 es mejor que servicios externos para imágenes 
●	Problema: Intentos con Pixabay y via.placeholder.com fallaron por CORS/DNS 
●	Solución: Usar S3 público desde el inicio (ya está en la infraestructura) 
●	Aplicación: Priorizar infraestructura existente antes de servicios externos 
Lección 9: Validación de URLs antes de renderizar 
●	Problema: URLs inválidas causaban errores en el frontend 
●	Solución: Lista de dominios inválidos y validación antes de crear <img> 
●	Aplicación: Validar datos en el frontend, no solo confiar en el backend 


2.	LECCIONES DE PROCESO 
Testing y Debugging 
Lección 10: Probar con datos reales, no solo casos ideales 
●	Problema: Código funcionaba en casos ideales pero fallaba con datos reales de DynamoDB 
●	Solución: Siempre probar con datos reales de producción (o al menos estructura real) 
●	Aplicación: Crear scripts de prueba que usen la misma estructura que producción 
Lección 11: Logs detallados son esenciales 
●	Problema: Difícil debuggear sin saber qué datos llegaban al frontend 
●	Solución: Agregar logs detallados: [PROGRESS], [PARSE_IMAGES], [WS] 
●	Aplicación: Logs estructurados con prefijos hacen debugging mucho más rápido 
Lección 12: Consola del navegador es tu mejor amigo 
●	Problema: Errores silenciosos que no se veían 
●	Solución: Siempre abrir consola (F12) durante desarrollo y pruebas 
●	Aplicación: Hacer debugging con consola abierta desde el inicio 
Deployment 
Lección 13: CloudFront invalidation tarda 1-5 minutos 
●	Problema: Cambios no se veían inmediatamente después de deploy 
●	Solución: Esperar 1-5 minutos y usar Ctrl+Shift+R para forzar recarga 
●	Aplicación: Informar a usuarios sobre tiempo de propagación 
Lección 14: Verificar deploy antes de asumir que funcionó 
●	Problema: Asumir que deploy funcionó sin verificar 
●	Solución: Verificar CloudFront invalidation status, revisar logs de Lambda 
●	Aplicación: Siempre verificar que el deploy realmente funcionó 
Documentación 
Lección 15: Documentar mientras desarrollas 
●	Problema: Olvidar detalles importantes después de días 
●	Solución: Documentar decisiones y problemas mientras los resuelves 
●	Aplicación: Mantener un archivo de notas mientras trabajas 


3.	ERRORES A EVITAR 
Errores Comunes 
Error 1: Asumir estructura de datos sin verificar 
●	Ejemplo: Asumir que overview.totalMessages existe cuando en realidad es overview.engagement.totalMessages 
●	Prevención: Siempre hacer console.log(JSON.stringify(data, null, 2)) para ver estructura real 
Error 2: Merge superficial de objetos anidados 
●	Ejemplo: merged = {**dict1, **dict2} sobrescribe objetos anidados completamente 
●	Prevención: Merge profundo nivel por nivel para objetos anidados 
Error 3: No parsear números de DynamoDB 
●	Ejemplo: if (totalMessages > 10) falla si totalMessages es string "10" 
● 	Prevención: Siempre 	parseInt()	 o 	int()	 al leer de DynamoDB
Error 4: Calcular estado en frontend 
●	Ejemplo: Calcular logros en JavaScript se pierde al recargar 
●	Prevención: Backend es fuente de verdad, frontend solo renderiza 
Error 5: No validar URLs antes de usar 
●	Ejemplo: Crear <img src="url-invalida"> causa errores 
●	Prevención: Validar dominio y formato antes de crear elementos DOM 
  
4.	MEJORES PRÁCTICAS 
Código 
Práctica 1: Verificar antes de afirmar 
●	Nunca asumir que un archivo/función existe sin verificar 
●	Usar read_file, grep, codebase_search antes de modificar 
Práctica 2: Cambios incrementales 
●	Hacer cambios pequeños y probar después de cada uno 
●	No hacer múltiples cambios grandes a la vez 
Práctica 3: Manejo de errores explícito 
●	No usar try {} catch {} vacío 
●	Loggear errores con contexto: console.error('[CONTEXT] Error:', error) 
Testing 
Práctica 4: Probar con datos reales 
●	No solo casos ideales 
●	Usar estructura de datos real de producción 
Práctica 5: Verificar logs después de cambios 
●	Revisar consola del navegador 
●	Revisar logs de Lambda en CloudWatch 
Deployment 
Práctica 6: Verificar deploy 
●	No asumir que funcionó 
●	Verificar CloudFront invalidation 
●	Probar en producción después de deploy 
Práctica 7: Documentar cambios importantes 
●	Documentar qué cambió y por qué 
●	Facilitar debugging futuro 


5.	OPTIMIZACIONES FUTURAS 
Técnicas 
Optimización 1: Tests automatizados 
●	Tests unitarios habrían detectado ~50% de los problemas 
●	Implementar suite de tests básica 
Optimización 2: Modularizar código 
●	app.js con 1650+ líneas es difícil de mantener 
●	Separar en módulos: audio, 3D, UI, network 
Optimización 3: CI/CD pipeline 
●	Deploy manual es propenso a errores 
●	Automatizar con GitHub Actions o similar 
Proceso 
Optimización 4: Revisión de arquitectura al inicio 
●	Revisar estructura completa antes de implementar 
●	Ahorraría ~6-8 horas de iteraciones 
Optimización 5: Servidor de desarrollo local 
●	Probar cambios sin deploy 
●	Hot reload para desarrollo más rápido 


6.	MÉTRICAS DE TIEMPO 
Tiempo Invertido 
●	Total: ~110-140 horas (~3-4 semanas full-time) 
●	Desarrollo: ~80-100 horas 
●	Correcciones/Iteraciones: ~19-25 horas 
●	Documentación: ~10-15 horas 
Tiempo Perdido (Evitable) 
●	Panel de progreso: ~8-10 horas (evitable: ~6-8 horas) 
●	Sistema de imágenes: ~6-8 horas (evitable: ~4-5 horas) 
●	CSS feedback: ~3-4 horas (evitable: ~2 horas) 
●	Total evitable: ~14-17 horas (~12% del tiempo total) 
Lección de Tiempo 
●	Tests unitarios habrían ahorrado ~50% del tiempo en correcciones 
●	Revisión de arquitectura al inicio habría ahorrado ~40% del tiempo perdido 


7.	RECOMENDACIONES PARA FUTUROS PROYECTOS 
Al Iniciar un Proyecto 
1. Revisar estructura de datos completa 
●	Hacer queries de ejemplo a la base de datos 
●	Documentar estructura real (no asumida) 
1. Diseñar arquitectura correcta desde el inicio 
●	¿Dónde se calcula el estado? (Backend) 
●	¿Dónde se renderiza? (Frontend) 
●	¿Dónde se persiste? (Base de datos) 
1. Configurar tests básicos 
●	Al menos tests de estructura de datos 
●	Tests de funciones críticas 
Durante el Desarrollo 
1. Probar con datos reales desde el inicio 
●	No esperar hasta el final para probar 
●	Crear datos de prueba que reflejen producción 
1. Logs estructurados desde el inicio 
●	Prefijos consistentes: [MODULE] Mensaje 
●	Facilita debugging 
1. Documentar decisiones importantes 
●	Por qué se tomó una decisión 
●	Qué alternativas se consideraron 
Al Hacer Deploy 
1. Verificar que funcionó ● 	No asumir éxito 
	● 	Probar en producción 
1. Documentar cambios ● Qué cambió 
●	Por qué cambió 
●	Cómo probarlo 
  
8.	HERRAMIENTAS Y RECURSOS ÚTILES 
Desarrollo 
●	Consola del navegador (F12): Esencial para debugging frontend 
●	CloudWatch Logs: Para debugging backend/Lambda 
●	AWS CLI: Para verificar estado de recursos 
Scripts Útiles 
● 	verificar_datos_dynamodb.py	: Verificar estructura de datos
	● 	Scripts de deploy automatizados: Ahorran tiempo y errores 
Documentación 
●	Documentar mientras desarrollas 
●	Mantener índice de documentación actualizado 
  
9.	CONCLUSIÓN 
Logros Principales 
●	Sistema de imágenes 100% funcional 
●	Panel de progreso funcionando 
●	Sistema de logros persistente 
●	Documentación completa 
Lecciones Clave 
1. Verificar antes de asumir: Estructura de datos, existencia de archivos, etc. 
1. Backend es fuente de verdad: No calcular estado en frontend 
1. Merge profundo para objetos anidados: Preservar datos existentes 
1. Probar con datos reales: No solo casos ideales 
1. Logs estructurados: Facilitan debugging enormemente 
Tiempo Ahorrable 
●	Con mejor proceso: ~14-17 horas (~12% del tiempo total) 
●	Con tests automatizados: ~50% del tiempo en correcciones 
●	Con revisión de arquitectura al inicio: ~40% del tiempo perdido 
Estado Final 
●	Funcional: ✅ Sí 
●	Estable: ⚠ Parcialmente (75%) 
●	Escalable: ❌ No (requiere refactorización) 
●	Mantenible: ⚠ Difícil (código monolítico) 
 
Última actualización: 5 de enero de 2026Próximo proyecto: Aplicar estas lecciones desde el día 1 
