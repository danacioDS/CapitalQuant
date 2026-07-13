# CapitalMind
#!/bin/bash
# CapitalMind - Git Update Script con manejo de errores

echo "📂 CapitalMind - Actualizando repositorio..."
echo ""

# 1. Ver estado actual
echo "🔍 Verificando estado..."
git status
echo ""

# 2. Añadir todos los archivos
echo "📦 Añadiendo archivos..."
git add .
echo ""

# 3. Hacer commit (con fecha y hora)
echo "📝 Haciendo commit..."
git commit -m "update $(date '+%Y-%m-%d %H:%M')"
echo ""

# 4. Intentar hacer push, si falla, hacer pull primero
echo "🚀 Subiendo a GitHub..."
if git push origin master; then
    echo "✅ ¡Actualización completada con éxito!"
else
    echo "⚠️ Push rechazado. Haciendo pull primero..."
    git pull origin master
    echo "📝 Intentando push nuevamente..."
    git push origin master
    echo "✅ ¡Actualización completada con éxito!"
fi

echo ""
echo "📊 Estado final:"
git status