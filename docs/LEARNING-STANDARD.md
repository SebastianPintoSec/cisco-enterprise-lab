# Estándar de ingeniería y aprendizaje

Cada práctica debe responder:

1. **Objetivo:** qué problema resolvemos y dónde aparece en producción.
2. **Fundamentos:** protocolos, capas OSI, tablas, planos de control y datos.
3. **Diseño:** topología, direccionamiento, interfaces, dependencias, riesgos.
4. **Configuración explicada:** sintaxis y finalidad de cada comando, impacto y reversión.
5. **Verificación:** resultado esperado, resultado real y evidencia.
6. **Troubleshooting:** síntoma, hipótesis, pruebas, causa raíz, solución y nueva validación.
7. **Seguridad:** privilegio mínimo, aislamiento, exposición y respaldo.
8. **Conclusiones:** aprendizajes, limitaciones y siguientes pasos.

Estados admitidos: PLANIFICADO, CONFIGURADO, VALIDADO, FALLÓ, COMPLETADO.

Las pruebas negativas (accesos que *deben fallar*) son parte de la validación. Nunca publicar secretos, claves privadas, contraseñas ni configuraciones operativas completas sin sanitizar.
