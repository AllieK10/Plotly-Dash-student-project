from dash import Dash, html, dcc
import plotly.express as px
import pandas as pd
from dash import dcc
from dash import html
from dash.dependencies import Input, Output
import dash_bootstrap_components as dbc
from dash import dash_table

app = Dash(__name__, external_stylesheets=[dbc.themes.UNITED], prevent_initial_callbacks=True)
markdown_text = '''
> _This project is all about bees. No specific reason, I just really like the beeline-design :D_

> So, here you can see the map with all the data considering the poor condition of bee colonies in the US (_which is kinda sad, actually_). 
And don't forget to try animation in order to see how everything's changed over the years!

_P.S: if you can't see the map, switch options in the dropdown below, it may be glitchy sometimes_
'''

text = '''
Here you can choose **STATE**, **NEGATIVE EFFECT** (even several of them), 
**PERCENT** and **YEAR** to filter the database by selected parameter(s).

_**Attention**_: when you use %-slider, you get interval between 0 and chosen %. 
Moreover, everything in here is codependent: whatever you select via sliders is automatically depicted in graphs.
_**HAVE FUN!:)**_ 
'''

final_words = '''
Last but not least: Pie chart. Choose states to count the percentage of bee colonies impacted.

_P.S: if pie chart is not displayed by default (although it has to be), click buttons._

> _**Hope everything turned out well, I enjoyed making this so much ^.^**_
'''


df = pd.read_csv("intro_bees.csv")
print(df.head())

state_indicators = df['State'].unique()
disease_indicators = df['Affected by'].unique()

mytitle = dcc.Markdown(children='# **My project**',
                       style={'text-align': 'center'})

mygraph = dcc.Graph(figure={})

dropdown = dcc.Dropdown(options=['Bar plot', 'Choromap'],
                         value='Choromap',
                         clearable=False)

row_drop = dcc.Dropdown(value=10, clearable=False, style={'width':'35%'},
                                 options=[10, 25, 50, 100])

mytable = dash_table.DataTable(
     columns=[
        {"name": i, "id": i, "deletable": True, "selectable": False, "hideable": True} for i in df.columns
     ],
     data=df.to_dict('records'),
     page_size=10,
     filter_action='native',
     style_as_list_view=True,
     sort_action="native",
     sort_mode="multi",
     style_header={'backgroundColor': 'black', 'color': '#e3c986', 'fontWeight': 'bold'},
     style_data={'backgroundColor': 'rgb(50, 50, 50)', 'color': 'white'},
     selected_rows=[],
     selected_columns=[],
     row_selectable="multi"
)

mycheckbox = dcc.Checklist(options=[{'label': x, 'value': x, 'disabled':False}
                           for x in df['state_code'].unique()],
                           value=['NY', 'CA', 'TX', 'WA', 'NE'],
                           inline=True,
                           style={'display':'flex'},
                           inputStyle={'cursor':'pointer'},

                           )

piechart = dcc.Graph()

app.layout = dbc.Container([
    mytitle, html.Br(), dcc.Markdown(children=markdown_text), html.Br(), dropdown, mygraph, html.Br(),
    dcc.Markdown(children=text),
    dbc.Row([
            dbc.Col([
                state_drop := dcc.Dropdown([x for x in sorted(state_indicators)])
            ], width=3),
            dbc.Col([
                disease_drop := dcc.Dropdown([x for x in sorted(disease_indicators)], multi=True)
            ], width=3),
            dbc.Col([
                pct_slider := dcc.Slider(0, 90, marks={'0':'0%', '90':'90%'},
                                         value=0, tooltip={"placement": "bottom", "always_visible": True})
            ], width=3),
            dbc.Col([
                year_slider := dcc.Slider(marks={2015:'2015', 2016:'2016', 2017:'2017', 2018:'2018', 2019:'2019'},
                                          value=2015, step=1,
                                       tooltip={"placement": "bottom", "always_visible": True})
            ], width=3),
            dbc.Col([row_drop], width=2)

        ], justify="between", className='mt-3 mb-4'),
    mytable, html.Br(), mycheckbox, piechart, dcc.Markdown(children=final_words)

])

@app.callback(
    Output(piechart, 'figure'),
    Input(mycheckbox, 'value')
)
def update_graph(options_chosen):
    dff = df[df['state_code'].isin(options_chosen)]
    print(dff['state_code'].unique())

    piechart = px.pie(
        data_frame=dff,
        names='State',
        values='Pct of Colonies Impacted',
        color_discrete_sequence=px.colors.sequential.YlOrRd,
        hole=.3,
        height=600,
        labels={'State': 'State'}
    )

    return piechart

@app.callback(
    Output(mytable, 'data'),
    Output(mytable, 'page_size'),
    Input(state_drop, 'value'),
    Input(disease_drop, 'value'),
    Input(pct_slider, 'value'),
    Input(year_slider, 'value'),
    Input(row_drop, 'value')
)
def update_dropdown_options(state_v, disease_v, pct_v, year_v, row_v):
    dff = df.copy()

    if state_v:
        dff = dff[dff['State']==state_v]
    if disease_v:
        dff = dff[dff['Affected by'].isin(disease_v)]

    dff = dff[(dff['Pct of Colonies Impacted'] <= pct_v)]
    dff = dff[(dff['Year'] == year_v)]

    return dff.to_dict('records'), row_v

@app.callback(
    Output(mygraph, 'figure'),
    Input(dropdown, 'value'),
    Input(year_slider, 'value'),
    Input(disease_drop, 'value'),
    Input(mytable, 'derived_virtual_data'),
    Input(mytable, 'derived_virtual_selected_rows'))

def update_graph(user_input, year_val, disease_val, all_rows_data, slctd_row_indices):


    if user_input == 'Bar plot':
         dff = df.copy()
         if year_val:
            dff = dff[dff['Year'] == year_val]
         if disease_val:
            dff = dff[dff['Affected by'].isin(disease_val)]
         fig = px.bar(
            data_frame=dff,
            x='State',
            y='Pct of Colonies Impacted',
            hover_data=['State', 'Pct of Colonies Impacted'],
            labels={'Pct of Colonies Impacted': '% of Bee Colonies'},
            template='plotly_dark',
            color='Pct of Colonies Impacted',
            #color='Year',
            #color_discrete_map={"2015": '#ffea80',"2016":'#f5c105',"2017":'#fa8d07',"2018":'#e05c14',"2019": '#d62718'}
            color_continuous_scale=px.colors.sequential.YlOrRd
        )
    elif user_input == 'Choromap':
        dff = pd.DataFrame(all_rows_data)

        borders = [5 if i in slctd_row_indices else 1
                   for i in range(len(dff))]

        fig = px.choropleth(
            data_frame=dff,
          #  data_frame=df,
            locationmode='USA-states',
            locations='state_code',
            scope="usa",
            color='Pct of Colonies Impacted',
            animation_frame='Year',
            hover_data=['State', 'Pct of Colonies Impacted'],
            color_continuous_scale=px.colors.sequential.YlOrRd,
            labels={'Pct of Colonies Impacted': '% of Bee Colonies'},
            template='plotly_dark',
            height=600
        ).update_traces(marker_line_width=borders)

    return fig

if __name__ == '__main__':
    app.run_server(debug=True)
