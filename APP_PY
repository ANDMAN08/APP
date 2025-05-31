#----------------------------------------------------------------------------------------------------------------------------------------
# Autores: José Carlos Cordón, Gunther Franke, Angel Paiz, Manuel Pérez
# Carné: 25, 25, 25121, 23597
# Fecha: 03 de junio de 2025
# Descripción: App ReciBot
# Curso: Algoritmos y Programación Básica Sección: 90
#----------------------------------------------------------------------------------------------------------------------------------------



#----------------------------------------------------------------------------------------------------------------------------------------
#--------------------------------------Importamos todas las librerías que necesita el código para funcionar------------------------------
#----------------------------------------------------------------------------------------------------------------------------------------
import pandas as pd #Pandas es para la permanencia de datos
import streamlit as st #Para que se comporte como una app
import os #Para interactuar con los archivos de la computadora
import matplotlib.pyplot as plt #Es para generar estadísticas y cálculos matemáticos necesarios en el código
import io #Para trabajar datos de streamlit al disco y escribirlos de mejor manera :)
import streamlit_survey as ss
from datetime import datetime, timedelta, date #Extraer fechas y situarse temporalmente


#----------------------------------------------------------------------------------------------------------------------------------------
#----------------------Funciones para calculos estadísticos y almacenamiento de datos sobre la basura------------------------------------
#----------------------------------------------------------------------------------------------------------------------------------------


# -------------------------Función 1: obtener los datos que se usarán para el formulario-------------------------------------------------

def obtener_datos_usuario_formulario(nombre_usuario: str, fecha_registro: date,
                                     organico: float, plastico: float, papel: float,
                                     vidrio: float, metal: float, no_reciclable: float) -> dict:
    return {
        "usuario": nombre_usuario,
        "fecha": str(fecha_registro),
        "organico": organico,
        "plastico": plastico,
        "papel": papel,
        "vidrio": vidrio,
        "metal": metal,
        "no_reciclable": no_reciclable
    }

# ----------------------Funcion 2: Sirve para procesar todos los datos que se estarán trabajando--------------------
def procesar_datos_basura(datos: dict, kg_por_bolsa: float = 3.0) -> tuple:
    tipos = ["organico", "plastico", "papel", "vidrio", "metal", "no_reciclable"]
    bolsas = {tipo: round(datos[tipo] / kg_por_bolsa, 2) for tipo in tipos}
    semanal = {tipo: datos[tipo] for tipo in tipos}
    mensual = {tipo: round(datos[tipo] * 4, 2) for tipo in tipos}
    anual = {tipo: round(datos[tipo] * 52, 2) for tipo in tipos}
    return bolsas, semanal, mensual, anual

# ----------------------Funcion 3: sirve para almacenar posteriormente los datos en un archivo CSV--------------------
def guardar_datos_en_csv(datos: dict, bolsas: dict,
                         semanal: dict, mensual: dict, anual: dict,
                         ruta_csv: str = "datos_basura.csv") -> pd.DataFrame:
    
    fila = {
        "Usuario": datos["usuario"],
        "Fecha": datos["fecha"],
        **{f"Bolsas_{k}": v for k, v in bolsas.items()},
        **{f"Semanal_{k}": v for k, v in semanal.items()},
        **{f"Mensual_{k}": v for k, v in mensual.items()},
        **{f"Anual_{k}": v for k, v in anual.items()}
    }

    if os.path.exists(ruta_csv):
        df_existente = pd.read_csv(ruta_csv)
        df_nuevo = pd.concat([df_existente, pd.DataFrame([fila])], ignore_index=True)
    else:
        df_nuevo = pd.DataFrame([fila])

    df_nuevo.to_csv(ruta_csv, index=False)
    return df_nuevo

# -----------------------Función 5: Sirve para recuperar los datos desde el CSV-------------------
def cargar_datos_csv(ruta_csv: str) -> pd.DataFrame:
    if not pd.io.common.file_exists(ruta_csv):
        return pd.DataFrame()
    return pd.read_csv(ruta_csv)

# ------------Función 6: Sirve para filtrar los datos y presentarlos de forma efectiva según la intencion del usuario-----------
def filtrar_datos(df: pd.DataFrame, usuario: str, fecha: str) -> pd.DataFrame:
    if usuario:
        df = df[df["Usuario"].str.contains(usuario, case=False)]
    if fecha:
        df = df[df["Fecha"] == fecha]
    return df

# ----------------------Funcion 7: sirve para crear la tabla solo con los datos filtrados--------------------
def obtener_tabla_filtrada(df: pd.DataFrame) -> pd.DataFrame:
    return df

# -------------Opcion 8: sirve para crear la gráfica con los datos de las bolsas (es una gráfica de barras)-----------------------------
def obtener_figura_bolsas(df: pd.DataFrame) -> plt.Figure:
    columnas_bolsas = [col for col in df.columns if col.startswith("Bolsas_")]
    suma_bolsas = df[columnas_bolsas].sum()

    fig, ax = plt.subplots()
    suma_bolsas.plot(kind='bar', ax=ax, color='darkgreen')
    ax.set_ylabel("Cantidad de bolsas")
    ax.set_xlabel("Tipo de residuo")
    ax.set_title("Total de bolsas usadas por tipo")

    return fig

# ---------------Opción 9: sirve para crear la figura de la distribucion de días (es de tipo pie)---------------------------
def obtener_figura_temporal(df: pd.DataFrame, periodo: str) -> plt.Figure | None:
    columnas = [col for col in df.columns if col.startswith(f"{periodo}_")]
    if not columnas:
        return None

    totales = df[columnas].sum()

    fig, ax = plt.subplots()
    totales.plot(kind='pie', autopct='%1.1f%%', ax=ax)
    ax.set_ylabel("")
    ax.set_title(f"Distribución de basura {periodo.lower()}")

    return fig

#---------------------------------------------------------------------------------------------------------------------------
#-----------------------------------------Aqui empieza el código de la interfaz---------------------------------------------
#---------------------------------------------------------------------------------------------------------------------------

#-------------------------------------Ruta base del script para hallar el logo y el page icon-------------------------------

BASE_DIR = os.path.dirname(os.path.abspath(__file__))
LOGO_PATH = os.path.join(BASE_DIR, "Logo.png")
ICONO_PATH = os.path.join(BASE_DIR, "app.ico")

#-----------------------------------------------Configuración de la página--------------------------------------------------

st.set_page_config(page_title="ReciBot", page_icon=ICONO_PATH, layout="centered")
# Footer or info section
st.markdown("---")  # horizontal separator line

last_update = datetime(2025, 5, 31, 10, 0)  # example last update time



#------------------------------------------------Mostrar logo si existe-----------------------------------------------------

if os.path.exists(LOGO_PATH):
    st.image(LOGO_PATH, width=1100)
else:
    st.warning("⚠️ No se encontró el archivo 'Logo.png'. Asegúrate de que esté en la misma carpeta que este script.")

#------------------------------------------------Variables contadoras------------------------------------------------------
organico = 0
reciclable = 0
no_reciclable = 0
papel = 0
especial = 0

basura = {
    "Orgánico": organico,
    "Reciclable": reciclable,
    "No Reciclable": no_reciclable,
    "Papel": papel,
    "Especial(Punto limpio)": especial
}

#-------------------------------------------------------Preguntas----------------------------------------------------------

Q1 = "♻️ ¿El objeto es un residuo sólido o líquido?"
Q2 = "🍎 ¿El objeto es parte de un alimento o proviene de algo natural (fruta, verdura, carne, huevo, pan, flores, hojas)?"
Q3 = "🔥 ¿Está cocinado o tiene grasa?"
Q4 = "📄 ¿Está hecho principalmente de papel o cartón?"
Q5 = "💧 ¿Tiene restos de comida, está mojado o tiene grasa?"
Q6 = "📃 ¿Es papel blanco, hojas impresas, libretas o cajas de cartón?"
Q7 = "🧴 ¿El residuo es de vidrio, plástico o metal?"
Q8 = "✨ ¿Está limpio y sin restos de comida o líquido?"
Q9 = "🔍 Tipo de material:"
Q10 = "🚼 ¿Se trata de objetos higiénicos o personales (pañales, toallas sanitarias, cotonetes, colillas)?"
Q11 = "👕 ¿Son textiles, ropa vieja o zapatos?"
Q12 = "📱 ¿Es un aparato electrónico, pila o foco?"
Q13 = "💊 ¿Es un medicamento, jeringa, o químico (pintura, aceite, etc.)?"
error = "❌ No es posible encontrar tu basura, deséchala en el centro de recuperación más cercano usando un contenedor sellado"

#--------------------------------------------------------------Opciones---------------------------------------------------------------------------

opciones = ["Bienvenida", "Formulario de clasificación", "Preguntas Frecuentes", "Ingresar basura para estadística", "mostrar basura"]
seccion = st.sidebar.selectbox("MENU", opciones)


#-----------------------------------------------------------Bienvenida----------------------------------------------------------------

if seccion == "Bienvenida":
    st.subheader("¡Ya puedes empezar a utilizar nuestra interfaz!")
    st.markdown(
        """
        <p style='font-size:18px; color: #2E86C1;'>
        Puedes empezar a usar la aplicación seleccionando alguna de las opciones en la barra lateral. 
        <br>¡Recuerda responder con sinceridad para obtener resultados precisos! 😊
        </p>
        """,
        unsafe_allow_html=True
    )
#----------------------------------------------------------Formulario----------------------------------------------------------------
elif seccion == "Formulario de clasificación":
    st.markdown("## 🗑️ Formulario de Clasificación de Residuos")
    st.markdown(
        """
        <p style='font-size:18px; color:#117A65;'>
        Por favor, responde las siguientes preguntas para clasificar correctamente tu basura. 
        </p>
        <p style='font-size:16px; color:#1F618D;'>
        Para todas las opciones, escribe el valor numérico correspondiente a la opción que deseas seleccionar. 🔢
        </p>
        """,
        unsafe_allow_html=True
    )

    survey = ss.StreamlitSurvey("Survey 1")

    Respuesta1 = survey.radio(f"🟢 {Q1}", options=["Sólido", "Líquido"])
    if Respuesta1 == "Sólido":
        Respuesta2 = survey.radio(f"🍎 {Q2}", options=["Sí", "No"])
        if Respuesta2 == "Sí":
            Respuesta3 = survey.radio(f"🍳 {Q3}", options=["Sí", "No"])
            if Respuesta3 == "Sí":
                st.markdown("### ✅ Deséchalo en: **Contenedor Orgánico** 🌱")
                organico += 1
            elif Respuesta3 == "No":
                SubRes3 = survey.radio("🌰 ¿Es cáscara, semilla, hueso o vegetal?", options=["Sí", "No"])
                if SubRes3 == "Sí":
                    st.markdown("### ✅ Deséchalo en: **Contenedor Orgánico** 🌿")
                    organico += 1
                elif SubRes3 == "No":
                    st.markdown(f"### ❌ {error} ⚠️")
        elif Respuesta2 == "No":
            Respuesta4 = survey.radio(f"📄 {Q4}", options=["Sí", "No"])
            if Respuesta4 == "Sí":
                Respuesta5 = survey.radio(f"🍔 {Q5}", options=["Sí", "No"])
                if Respuesta5 == "Sí":
                    st.markdown("### 🚫 Deséchalo en: **Contenedor No Reciclable** 🗑️")
                    no_reciclable += 1
                elif Respuesta5 == "No":
                    Respuesta6 = survey.radio(f"📚 {Q6}", options=["Sí", "No"])
                    if Respuesta6 == "Sí":
                        st.markdown("### ♻️ Deséchalo en: **Contenedor de Papel** 📦")
                        papel += 1
                    elif Respuesta6 == "No":
                        SubRes6 = survey.radio("🧻 ¿Es papel encerado, plastificado o papel higiénico?", options=["Sí", "No"])
                        if SubRes6 == "Sí":
                            st.markdown("### 🚫 Deséchalo en: **Contenedor No Reciclable** 🗑️")
                            no_reciclable += 1
                        elif SubRes6 == "No":
                            st.markdown(f"### ❌ {error} ⚠️")
            elif Respuesta4 == "No":
                Respuesta7 = survey.radio(f"🧴 {Q7}", options=["Sí", "No"])
                if Respuesta7 == "Sí":
                    Respuesta8 = survey.radio(f"🧼 {Q8}", options=["Sí", "No"])
                    if Respuesta8 == "Sí":
                        Respuesta9 = survey.radio(f"🔍 {Q9}", options=[
                            "Plástico PET (botellas, envases)",
                            "Vidrio (botellas, frascos sin tapa)",
                            "Latas de aluminio",
                            "Plástico duro, mezclado o bolsas sucias"
                        ])
                        if Respuesta9 in ["Plástico PET (botellas, envases)", "Vidrio (botellas, frascos sin tapa)", "Latas de aluminio"]:
                            st.markdown("### ♻️ Deséchalo en: **Contenedor Reciclable** 🔄")
                            reciclable += 1
                        elif Respuesta9 == "Plástico duro, mezclado o bolsas sucias":
                            st.markdown("### 🚫 Deséchalo en: **Contenedor No Reciclable** 🗑️")
                            no_reciclable += 1
                    elif Respuesta8 == "No":
                        st.markdown("### 🚫 Deséchalo en: **Contenedor No Reciclable** (A menos que se lave antes) 🧼")
                        no_reciclable += 1
                elif Respuesta7 == "No":
                    Respuesta10 = survey.radio(f"🧻 {Q10}", options=["Sí", "No"])
                    if Respuesta10 == "Sí":
                        st.markdown("### 🚫 Deséchalo en: **Contenedor No Reciclable** 🗑️")
                        no_reciclable += 1
                    elif Respuesta10 == "No":
                        Respuesta11 = survey.radio(f"👕 {Q11}", options=["Sí", "No"])
                        if Respuesta11 == "Sí":
                            st.markdown("### 🚫 Deséchalo en: **Contenedor No Reciclable** (Si es posible, llévalo a un punto de reciclaje textil o, si están rotos o sucios) 🧺")
                            no_reciclable += 1
                        elif Respuesta11 == "No":
                            Respuesta12 = survey.radio(f"🔋 {Q12}", options=["Sí", "No"])
                            if Respuesta12 == "Sí":
                                st.markdown("### ⚠️ No se debe tirar en contenedores comunes. Llévalo a un punto limpio o reciclaje electrónico 🔌")
                                especial += 1
                            elif Respuesta12 == "No":
                                Respuesta13 = survey.radio(f"💊 {Q13}", options=["Sí", "No"])
                                if Respuesta13 == "Sí":
                                    st.markdown("### ⚠️ Punto limpio o farmacia autorizada. Nunca en contenedor común. 🏥")
                                    especial += 1
                                elif Respuesta13 == "No":
                                    st.markdown(f"### ❌ {error} ⚠️")
    elif Respuesta1 == "Líquido":
        st.markdown("### ⚠️ NO debe desecharse en contenedor común. Llévalo a un punto limpio especializado. 🚱")
        especial += 1

#-----------------------------------------------------------Preguntas Frecuentes----------------------------------------------------------------

elif seccion == "Preguntas Frecuentes":
    st.title("Preguntas Frecuentes - Clasificación de Basura (Guatemala 2025)")
    st.markdown("Selecciona una pregunta para ver su respuesta:")

    # Diccionario de preguntas y respuestas
    preguntas_respuestas = {
        "1. ¿Qué tipos de residuos se deben clasificar en Guatemala?":
            "Orgánico, Reciclable, No Reciclable, Papel y Especial (Punto limpio).",
        "2. ¿Qué va en el contenedor orgánico?":
            "Restos de alimentos naturales como frutas, verduras, cáscaras, semillas, huesos, pan, etc.",
        "3. ¿Qué residuos son reciclables?":
            "Plástico PET, vidrio limpio, latas de aluminio y papel o cartón limpio.",
        "4. ¿El papel mojado se puede reciclar?":
            "No, debe ir al contenedor No Reciclable.",
        "5. ¿Dónde va el papel blanco o impreso limpio?":
            "Contenedor de Papel.",
        "6. ¿Qué hago con pañales y toallas sanitarias?":
            "Van en el contenedor No Reciclable.",
        "7. ¿Y con ropa vieja o textiles?":
            "Preferiblemente llevarlos a reciclaje textil. Si están sucios o rotos, van en el No Reciclable.",
        "8. ¿Dónde van electrónicos, pilas y focos?":
            "Punto limpio o reciclaje electrónico autorizado.",
        "9. ¿Cómo desechar medicamentos vencidos o jeringas?":
            "Llevar a farmacias autorizadas o puntos limpios.",
        "10. ¿Qué hacer con pintura o aceite usado?":
            "Desechar en puntos limpios. Nunca en el drenaje o contenedor común.",
        "11. ¿Las bolsas plásticas se reciclan?":
            "Solo si están limpias y secas.",
        "12. ¿Qué hago con empaques de comida rápida?":
            "Si tienen grasa o residuos, van al No Reciclable.",
        "13. ¿Dónde van los empaques tipo tetrapack?":
            "Limpios y secos pueden reciclarse. Sucios, al No Reciclable.",
        "14. ¿Las botellas de plástico son siempre reciclables?":
            "Sí, si están limpias, secas y vacías.",
        "15. ¿Qué pasa si mezclo residuos?":
            "Contaminas materiales reciclables y dificultas su aprovechamiento.",
        "16. ¿Debo enjuagar los reciclables?":
            "Sí, siempre deben estar limpios y secos.",
        "17. ¿Qué es un punto limpio?":
            "Centro especializado para residuos peligrosos o electrónicos.",
        "18. ¿El papel higiénico es reciclable?":
            "No. Va en el No Reciclable.",
        "19. ¿Dónde reporto un punto limpio dañado?":
            "Municipalidad o Ministerio de Ambiente.",
        "20. ¿Las empresas deben clasificar basura?":
            "Sí, es obligatorio según la normativa 2025.",
        "21. ¿Dónde van los utensilios de madera como palillos o paletas?":
            "Si no están sucios, pueden ir al contenedor Orgánico. Si tienen residuos, al No Reciclable.",
        "22. ¿Dónde tiro los cepillos de dientes?":
            "Contenedor No Reciclable o reciclaje especializado si es de bambú o reciclable.",
        "23. ¿Las servilletas usadas se reciclan?":
            "No. Van en el contenedor No Reciclable.",
        "24. ¿Qué hago con papel aluminio?":
            "Limpio, puede reciclarse. Sucio, No Reciclable.",
        "25. ¿Dónde van los vasos de cartón encerado?":
            "Al contenedor No Reciclable.",
        "26. ¿Puedo reciclar CDs o DVDs?":
            "No en el reciclaje tradicional. Pueden llevarse a puntos limpios si se aceptan.",
        "27. ¿Qué hago con las cajas de pizza?":
            "Partes limpias van al contenedor de Papel. Las grasosas, al No Reciclable.",
        "28. ¿Los juguetes rotos se reciclan?":
            "En general no. Van al No Reciclable, salvo excepciones si están limpios y son de plástico.",
        "29. ¿Cómo descartar esponjas de cocina?":
            "Van en el contenedor No Reciclable.",
        "30. ¿Qué pasa si tiro residuos peligrosos en la basura común?":
            "Contaminas el medio ambiente y puedes causar accidentes.",
        "31. ¿Los cubiertos plásticos son reciclables?":
            "Solo si están limpios y si el centro acepta ese tipo de plástico.",
        "32. ¿Dónde van las botellas de vidrio rotas?":
            "En el reciclaje de vidrio, bien protegidas y limpias.",
        "33. ¿Dónde tiro el aceite de cocina usado?":
            "Debe almacenarse en botellas cerradas y llevarse a un punto limpio.",
        "34. ¿Cómo clasificar residuos de jardín?":
            "Hojas, ramas pequeñas y flores van en Orgánico.",
        "35. ¿Dónde tiro materiales de construcción como cemento o yeso?":
            "Deben gestionarse como residuos especiales en puntos autorizados.",
        "36. ¿Los sorbetes (popotes) son reciclables?":
            "No, en general van en el contenedor No Reciclable.",
        "37. ¿Qué hacer con termos, floreros o cerámica rota?":
            "Van en el contenedor No Reciclable o en desechos especiales.",
        "38. ¿Cómo se clasifican las tapas plásticas?":
            "Reciclables si están limpias. Se recomienda separarlas.",
        "39. ¿Las cajas de huevo se reciclan?":
            "Sí, si son de cartón seco. Las de espuma van al No Reciclable.",
        "40. ¿Qué hago con cables o cargadores dañados?":
            "Llevar a reciclaje electrónico o punto limpio.",
        "41. ¿Los plumones y marcadores se reciclan?":
            "No. Van en el contenedor No Reciclable.",
        "42. ¿Las botellas con etiquetas se pueden reciclar?":
            "Sí. Se recomienda quitar la etiqueta si es posible.",
        "43. ¿Cómo clasifico los envoltorios de golosinas?":
            "Van en el contenedor No Reciclable.",
        "44. ¿Se reciclan los envases de yogurt?":
            "Sí, si están completamente limpios y secos.",
        "45. ¿Dónde tiro el cartón de huevo mojado?":
            "Va al contenedor No Reciclable.",
        "46. ¿Qué hago con los botes de desodorante en aerosol?":
            "Punto limpio. Son residuos presurizados y peligrosos.",
        "47. ¿Puedo reciclar frascos de vidrio con tapa?":
            "Sí, pero es mejor separar la tapa (metal/plástico).",
        "48. ¿Qué es reciclaje mixto?":
            "Es cuando varios materiales reciclables se recogen juntos para su posterior separación.",
        "49. ¿Puedo reciclar botellas con tapas?":
            "Sí. Si el centro las acepta así, incluso mejor.",
        "50. ¿Dónde encuentro los centros de reciclaje en Guatemala?":
            "Consulta en el portal del Ministerio de Ambiente o municipalidades locales."
    }

    # Clave persistente: número de preguntas mostradas
    if "preguntas_mostradas" not in st.session_state:
        st.session_state.preguntas_mostradas = 10  # mostrar primeras 10

    # Lista de todas las preguntas
    lista_preguntas = list(preguntas_respuestas.items())

    # Mostrar preguntas actuales
    for i in range(st.session_state.preguntas_mostradas):
        pregunta, respuesta = lista_preguntas[i]
        with st.expander(pregunta):
            st.write(respuesta)

    # Botón para mostrar más
    if st.session_state.preguntas_mostradas < len(lista_preguntas):
        if st.button("🔽 Ver más preguntas"):
            st.session_state.preguntas_mostradas += 5

#-----------------------------------------------------------Ingreso de datos de basura----------------------------------------------------------------

elif seccion == "Ingresar basura para estadística":
    st.subheader("Rellene los datos siguientes:")

    nombre_usuario = st.text_input("Nombre del usuario")
    fecha_registro = st.date_input("Fecha del registro", value=date.today())

    st.markdown("### Ingrese la cantidad de basura generada (en kilogramos):")

    organico = st.number_input("Orgánica", min_value=0.0, step=0.1)
    plastico = st.number_input("Plástico", min_value=0.0, step=0.1)
    papel = st.number_input("Papel y cartón", min_value=0.0, step=0.1)
    vidrio = st.number_input("Vidrio", min_value=0.0, step=0.1)
    metal = st.number_input("Metal", min_value=0.0, step=0.1)
    no_reciclable = st.number_input("No reciclable", min_value=0.0, step=0.1)

    if st.button("Guardar información"):
        if nombre_usuario.strip() == "":
            st.warning("⚠️ Debe ingresar un nombre.")
        else:
            datos_usuario = obtener_datos_usuario_formulario(
                nombre_usuario, fecha_registro,
                organico, plastico, papel,
                vidrio, metal, no_reciclable
            )

            bolsas, semanal, mensual, anual = procesar_datos_basura(datos_usuario)

            df_resultado = guardar_datos_en_csv(
                datos_usuario, bolsas, semanal, mensual, anual
            )

            st.success("✅ Datos guardados correctamente.")
            st.dataframe(df_resultado.tail(1))  # Mostrar último registro guardado
    
#-----------------------------------------------------------Análisis estadístico----------------------------------------------------------------

elif seccion == "mostrar basura":
    st.header("📊 Visualización de Datos Recopilados")

    ruta_csv = "datos_basura.csv"
    df = cargar_datos_csv(ruta_csv)

    if df.empty:
        st.warning("⚠️ Aún no hay datos almacenados.")
    else:
        usuarios = sorted(df["Usuario"].unique())
        fechas = sorted(df["Fecha"].unique())

        usuario_filtro = st.selectbox("🔍 Buscar por usuario", options=[""] + usuarios)
        fecha_filtro = st.selectbox("📅 Buscar por fecha", options=[""] + fechas)

        df_filtrado = filtrar_datos(df, usuario_filtro, fecha_filtro)

        if df_filtrado.empty:
            st.info("No se encontraron registros con esos filtros.")
        else:
            tabla = obtener_tabla_filtrada(df_filtrado)
            st.markdown("### 📋 Datos filtrados:")
            st.dataframe(tabla)

            figura_bolsas = obtener_figura_bolsas(df_filtrado)
            st.markdown("### 🛍️ Cantidad de bolsas por tipo de basura")
            st.pyplot(figura_bolsas)

            for periodo in ["Semanal", "Mensual", "Anual"]:
                figura_temporal = obtener_figura_temporal(df_filtrado, periodo)
                if figura_temporal:
                    st.markdown(f"### 📈 Distribución de basura {periodo.lower()}")
                    st.pyplot(figura_temporal)


st.markdown("""
<div style="font-size: 0.8em; color: gray;">
    Developed by: UVG Students <br>
    Last updated: {update}<br>
    © 2025 All rights reserved.
</div>
""".format(update=last_update.strftime("%Y-%m-%d %H:%M")), unsafe_allow_html=True)
