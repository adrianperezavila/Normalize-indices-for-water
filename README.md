# Normalize-indices-for-water
Geochemical normalization indices with the Eh-pH behaviour of potentially hazardous elements in environmental contexts. The objective is to interpret mobility, fixation, and environmental risk based on geochemical processes, not just concentrations.
Example for potentially hazardous elements (As, Hg, Cd, Se, Li, U, P, Pb, Cu, Zn, Ni, Sb, Cr, Co, Ca, Al, K, Mg, Tl, Al, Fe, Mn): http://weppi.gtk.fi/publ/foregsatlas/articles.php). But applicable to mineral exploration.


import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

# -------------------------------
# 1. Cargar datos desde Excel
# cambian nombre del archivo: Datos agua Clean imputados Fase 2 para SIG.xlsx por nombre del archivo nuevo
# -------------------------------
file_path = 'Datos agua Clean imputados Fase 2 para SIG.xlsx'
data = pd.read_excel(file_path)

# -------------------------------
# 2. Columnas correctas con sufijo de unidad
# -------------------------------
codigo_col = 'CODIGO_MUESTRA'

# Columnas en ppm
ppm_cols = ['Ca_ppm', 'Mg_ppm', 'Na_ppm', 'K_ppm', 'Al_ppm']
# Columnas en ppb
pbb_cols = ['As_ppb','Hg_ppb','Cd_ppb','Se_ppb','Li_ppb','U_ppb',
            'Pb_ppb','Cu_ppb','Zn_ppb','Ni_ppb','Sb_ppb','Cr_ppb',
            'Co_ppb','Tl_ppb','Fe_ppb','Mn_ppb','Si_ppb','SO4_ppb', 'P_ppb'] # Agregué P_ppb

# Convertir ppm → µg/L
for col in ppm_cols:
    if col in data.columns:
        data[col] = data[col] * 1000
# ppb ya están en µg/L
for col in ppb_cols:
    if col in data.columns:
        data[col] = data[col]

# -------------------------------
# 3. Calcular índices normalizados de forma robusta
# -------------------------------
def safe_div(numerator, denominator):
    try:
        if denominator == 0 or pd.isna(denominator):
            return np.nan
        else:
            return numerator / denominator
    except:
        return np.nan

def calcular_indices(row):
    indices = {}

    # As/(Ca+Na)
    if 'As_ppb' in row and 'Ca_ppm' in row and 'Na_ppm' in row:
        indices['As/(Ca+Na)'] = safe_div(row['As_ppb'], row['Ca_ppm'] + row['Na_ppm'])

    # Hg/(Ca+Mg)
    if 'Hg_ppb' in row and 'Ca_ppm' in row and 'Mg_ppm' in row:
        indices['Hg/(Ca+Mg)'] = safe_div(row['Hg_ppb'], row['Ca_ppm'] + row['Mg_ppm'])

    # Cd/Zn
    if 'Cd_ppb' in row and 'Zn_ppb' in row:
        indices['Cd/Zn'] = safe_div(row['Cd_ppb'], row['Zn_ppb'])

    # Se/SO4
    if 'Se_ppb' in row and 'SO4_ppb' in row:
        indices['Se/SO4'] = safe_div(row['Se_ppb'], row['SO4_ppb'])
    
    # Li/Na
    if 'Li_ppb' in row and 'Na_ppm' in row:
        indices['Li/Na'] = safe_div(row['Li_ppb'], row['Na_ppm'])
    
    # U/(Ca+Mg)
    if 'U_ppb' in row and 'Ca_ppm' in row and 'Mg_ppm' in row:
        indices['U/(Ca+Mg)'] = safe_div(row['U_ppb'], row['Ca_ppm'] + row['Mg_ppm'])
    
    # P/Ca
    if 'P_ppb' in row and 'Ca_ppm' in row:
        indices['P/Ca'] = safe_div(row['P_ppb'], row['Ca_ppm'])
    
    # Pb/Al
    if 'Pb_ppb' in row and 'Al_ppm' in row:
        indices['Pb/Al'] = safe_div(row['Pb_ppb'], row['Al_ppm'])
    
    # Cu/(Ca+Na)
    if 'Cu_ppb' in row and 'Ca_ppm' in row and 'Na_ppm' in row:
        indices['Cu/(Ca+Na)'] = safe_div(row['Cu_ppb'], row['Ca_ppm'] + row['Na_ppm'])
    
    # Zn/Ca
    if 'Zn_ppb' in row and 'Ca_ppm' in row:
        indices['Zn/Ca'] = safe_div(row['Zn_ppb'], row['Ca_ppm'])
    
    # Ni/Mg
    if 'Ni_ppb' in row and 'Mg_ppm' in row:
        indices['Ni/Mg'] = safe_div(row['Ni_ppb'], row['Mg_ppm'])
    
    # Sb/As
    if 'Sb_ppb' in row and 'As_ppb' in row:
        indices['Sb/As'] = safe_div(row['Sb_ppb'], row['As_ppb'])
    
    # Cr/Fe
    if 'Cr_ppb' in row and 'Fe_ppb' in row:
        indices['Cr/Fe'] = safe_div(row['Cr_ppb'], row['Fe_ppb'])
    
    # Co/Fe
    if 'Co_ppb' in row and 'Fe_ppb' in row:
        indices['Co/Fe'] = safe_div(row['Co_ppb'], row['Fe_ppb'])
    
    # Tl/K
    if 'Tl_ppb' in row and 'K_ppm' in row:
        indices['Tl/K'] = safe_div(row['Tl_ppb'], row['K_ppm'])
    
    # Fe/Mn
    if 'Fe_ppb' in row and 'Mn_ppb' in row:
        indices['Fe/Mn'] = safe_div(row['Fe_ppb'], row['Mn_ppb'])
    
    # Ca/(Na+K)
    if 'Ca_ppm' in row and 'Na_ppm' in row and 'K_ppm' in row:
        indices['Ca/(Na+K)'] = safe_div(row['Ca_ppm'], row['Na_ppm'] + row['K_ppm'])
    
    # Mg/Ca
    if 'Mg_ppm' in row and 'Ca_ppm' in row:
        indices['Mg/Ca'] = safe_div(row['Mg_ppm'], row['Ca_ppm'])
    
    # Al/Si
    if 'Al_ppm' in row and 'Si_ppb' in row:
        indices['Al/Si'] = safe_div(row['Al_ppm'], row['Si_ppb'])
        
    return indices

# Aplicar la función y expandir a nuevas columnas
indices_df = data.apply(calcular_indices, axis=1, result_type='expand')

# Insertar la columna de código de muestra al principio
indices_df.insert(0, codigo_col, data[codigo_col])

# -------------------------------
# 4. Definir colores para la movilidad Eh–pH
# -------------------------------
ehph_colors = {
    'As/(Ca+Na)': 'red', 'Hg/(Ca+Mg)': 'orange', 'Cd/Zn': 'red',
    'Se/SO4': 'red', 'Li/Na': 'green', 'U/(Ca+Mg)': 'red',
    'P/Ca': 'orange', 'Pb/Al': 'green', 'Cu/(Ca+Na)': 'red',
    'Zn/Ca': 'red', 'Ni/Mg': 'orange', 'Sb/As': 'orange',
    'Cr/Fe': 'red', 'Co/Fe': 'green', 'Tl/K': 'red',
    'Fe/Mn': 'orange', 'Ca/(Na+K)': 'green', 'Mg/Ca': 'green',
    'Al/Si': 'green'
}

# -------------------------------
# 5. Generar y mostrar gráficos de radar (radar plots)
# -------------------------------
labels = list(indices_df.columns[1:])
num_vars = len(labels)
angles = np.linspace(0, 2 * np.pi, num_vars, endpoint=False).tolist()

for i, row in indices_df.iterrows():
    values = row[labels].tolist()
    values += values[:1]  # Cerrar el polígono en el gráfico de radar
    plot_angles = angles + angles[:1]

    fig, ax = plt.subplots(figsize=(12,12), subplot_kw=dict(polar=True))

    # Trazar cada elemento individualmente con su color de movilidad
    for j, label in enumerate(labels):
        val = [values[j], values[j]]
        ang = [plot_angles[j], plot_angles[j+1]]
        ax.plot(ang, val, color=ehph_colors[label], linewidth=3)

    # Relleno poligonal general
    ax.fill(plot_angles, values, color='red', alpha=0.2)

    ax.set_xticks(angles)
    ax.set_xticklabels(labels, fontsize=10)
    ax.set_yticklabels([]) # Ocultar las etiquetas del eje radial
    ax.set_title(f"Índices de Contaminación con Eh–pH - {row['CODIGO_MUESTRA']}", size=14, pad=20)
    plt.show()

# -------------------------------
# 6. Guardar índices normalizados
# -------------------------------
indices_df.to_csv('indices_normalizados_agua_Eh_pH.csv', index=False)
