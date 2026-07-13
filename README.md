# 1. Ver el estado actual
git status

# 2. Ver las diferencias entre local y remoto
git log --oneline --graph --all

# 3. Configurar pull con merge (recomendado)
git config pull.rebase false

# 4. Hacer pull para traer cambios del remoto
git pull origin master

# 5. Si hay conflictos, resolverlos con:
#    - Editar los archivos conflictivos
#    - git add <archivo_resuelto>
#    - git commit -m "merge conflicts resolved"

# 6. Subir los cambios
git push origin master

# 7. Verificar que todo está sincronizado
git status