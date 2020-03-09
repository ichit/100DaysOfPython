# Chpater 17. Representation Learning and Generative Learning Using Autoencoders and GANs

Autoencoders (AE) are ANN capable of dense representations of the input data (*latent variables*) without supervision.
These codings are usually represented in lower dimensions, making AE powerful tools for dimensionality reduction for visualization and feature detectors for creating other models.

Generative Adversarial Networks (GAN) consist of two ANN: a *generator* that tries to create data that looks simillar to the training data, and a *discriminator* that tries to tell the real data from the fake data.
This architecture and training method are incredibly powerful and can create output that is incredibly realistic.

This chapter will explore both AE and GAN.


```python
import numpy as np
import pandas as pd 
import matplotlib as mpl
import matplotlib.pyplot as plt
import seaborn as sns
import plotly.express as px
import tensorflow as tf 
import tensorflow.keras as keras

%matplotlib inline
np.random.seed(0)
sns.set_style('whitegrid')
```


```python
def plot_train_history(h):
    pd.DataFrame(h.history).plot(figsize=(8, 5))
    plt.show()
```

## Efficient data representations

An AE must be constrained so that it is forced to learn patterns and not just memorize all of the inputs.
It is composed of two pieces, an *encoder* that converts the inputs to its learned latent representation and a *decoder* that converts the internal representation to the outputs.
(The encoder and decoder are also referred to as the *recognition network* and *generative network*.)

Interestingly, a simple AE looks just like a multi-layer perceptron (MLP) except that the number of inputs must be the same as the number of outputs.
Because there are fewer internal dimensions than the input or output, an AE is said to be *undercomplete*.
This means it cannot just copy and spit out the inputs, but instead must learn the most important features in the input data.

The output is often called the *reconstruction* and the loss function often includes the *reconstruction error*, how different was the output from the input.

## Performing PCA with an undercomplete linear autoencoder

If the AE uses only linear activation functions and the cost function is MSE, it ends up just performing PCA.


```python
from sklearn.datasets import make_blobs
from sklearn.model_selection import train_test_split

np.random.seed(20)
X, y = make_blobs(300, n_features=3)
X_train, X_test, y_train, y_test = train_test_split(X, y)

X_df = pd.DataFrame(X, columns=('x', 'y', 'z'))
X_df['blob'] = y.astype(str)
fig = px.scatter_3d(X_df, x='x', y='y', z='z', color='blob', 
                    size=[1] * len(X_df), size_max=13, opacity=0.7,
                    width=600, height=400, template='plotly_white')
fig.update_layout(margin=dict(l=0, r=0, b=0, t=0))
```


        <script type="text/javascript">
        window.PlotlyConfig = {MathJaxConfig: 'local'};
        if (window.MathJax) {MathJax.Hub.Config({SVG: {font: "STIX-Web"}});}
        if (typeof require !== 'undefined') {
        require.undef("plotly");
        define('plotly', function(require, exports, module) {
            /**
* plotly.js v1.52.1
* Copyright 2012-2020, Plotly, Inc.
* All rights reserved.
* Licensed under the MIT license
*/
        });
        require(['plotly'], function(Plotly) {
            window._Plotly = Plotly;
        });
        }
        </script>




<div>


            <div id="46c6c27a-3afd-4c74-b2c6-caa4f689d6c0" class="plotly-graph-div" style="height:400px; width:600px;"></div>
            <script type="text/javascript">
                require(["plotly"], function(Plotly) {
                    window.PLOTLYENV=window.PLOTLYENV || {};

                if (document.getElementById("46c6c27a-3afd-4c74-b2c6-caa4f689d6c0")) {
                    Plotly.newPlot(
                        '46c6c27a-3afd-4c74-b2c6-caa4f689d6c0',
                        [{"hoverlabel": {"namelength": 0}, "hovertemplate": "blob=1<br>x=%{x}<br>y=%{y}<br>z=%{z}<br>size=%{marker.size}", "legendgroup": "1", "marker": {"color": "#636efa", "opacity": 0.7, "size": [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1], "sizemode": "area", "sizeref": 0.005917159763313609, "symbol": "circle"}, "mode": "markers", "name": "1", "scene": "scene", "showlegend": true, "type": "scatter3d", "x": [7.365406229597365, 6.364556423512518, 7.683641698887902, 6.722347644531004, 6.748200357209985, 7.186421157501112, 6.416524736450471, 6.167359489631386, 5.243949945195828, 4.301919196844511, 7.47627457791353, 7.457162172828183, 4.437730313975638, 6.405389399548496, 4.631916321488295, 6.723398699303686, 7.327594399665015, 6.938021838442939, 6.742542068219035, 6.421510504586278, 5.117671191893014, 6.264583406531888, 5.386223239634285, 7.3174142642075095, 5.377327377580519, 8.053430201464744, 3.9198948524788073, 5.695663147544233, 5.437353126699372, 6.28392728440054, 5.3940096121664425, 7.308830285006497, 6.739748270814357, 7.400409804689243, 5.855863491941683, 6.235278387932875, 8.321021307624688, 6.7158847068875644, 6.815310527061523, 5.394966995522166, 7.206597527281165, 6.2971150510168945, 5.3370255632491945, 6.376052812178913, 5.652694447017575, 7.301173144284847, 6.302184379912232, 7.195537267014347, 4.9892157967473025, 6.758695945955199, 7.326875669380498, 6.82615769221623, 5.681379983809881, 5.590915872911531, 6.351209510652807, 7.259788244615354, 6.0612231207929135, 6.052763157593785, 6.491734523745514, 6.342859787533829, 4.965925353168621, 6.155411928318515, 6.0435196202742665, 5.5792036520356225, 7.3407316551042125, 6.377659389421145, 6.396397572180984, 7.082772580532854, 7.031554379125225, 6.603167358411424, 6.581399128322183, 6.859438850710169, 8.03240491703545, 6.533569280893025, 6.594487843181758, 5.73060363215336, 6.403109061562517, 7.568340135182062, 6.692713205756386, 5.043518435443536, 6.809028684754608, 6.495708265147161, 7.24489459046886, 7.354227084450656, 5.622066355555211, 4.794269860082657, 8.473764208901246, 7.545979575083751, 6.8231384348064825, 4.680496147378217, 6.858361484494765, 5.4770649158452525, 5.505669950474198, 6.261570553144677, 5.844822234962823, 5.222523729570874, 6.144182268875463, 7.190612622774797, 5.132974279812812, 5.39203499222556], "y": [-9.212585449092318, -9.932676875687287, -8.288675767257532, -8.36445865647428, -11.063929462449028, -10.007798863059392, -10.100498139676448, -8.975203936388594, -9.590906672604204, -8.865967584010777, -10.258974561644187, -8.375140071175014, -10.222374887297004, -9.609475656637128, -10.37536631104934, -9.2683481064154, -9.346894006711327, -8.863108630239427, -9.17179822983528, -9.128799558276079, -11.227244483369455, -9.195214781295267, -10.326909886351475, -8.798504480021318, -11.199050445698381, -9.497696791926638, -10.91907018089044, -9.682318129441567, -10.159230537744204, -8.882943224461924, -9.837485471870988, -7.7415666083203725, -10.701993634388664, -8.463839181970762, -11.017170741387515, -10.19104695173787, -9.813827787532986, -10.203951395922957, -8.781388802126829, -8.788048985331072, -9.729052486774204, -9.672515206023384, -9.522759639735451, -9.266279549297492, -9.400965778135298, -8.936975269243213, -8.632739293574502, -8.446399294481575, -10.03113619164781, -8.145288390412347, -7.52584666891531, -9.331390553146067, -8.574989128151781, -10.290359450536698, -9.59521693492132, -8.306842517394147, -11.425665190512392, -10.37990957301977, -9.927241277390396, -8.700414837165813, -8.95997914919921, -8.852709267082636, -8.387773583213852, -9.215807289252377, -9.56061445568493, -7.841527270052035, -9.594053743667821, -9.968512764112331, -8.189154632755924, -9.015170282087732, -10.468689802554826, -9.621380745465103, -8.867062828841997, -9.44651734150577, -10.32605109353307, -9.655613837772066, -10.399325293191977, -9.868222360784392, -7.807273751028235, -10.124156357416773, -9.78175413203175, -10.05112021940081, -10.017600342902798, -10.626580294722348, -9.10220838097228, -7.640706029631641, -8.386875971683944, -8.494557531669068, -10.190028850374455, -9.401077602247726, -7.9741537428523115, -10.645889107895314, -10.897182428211504, -9.146031622966452, -8.994367943268022, -7.918241643301086, -9.35325341522907, -10.699048979258531, -9.31987611759365, -8.945917950516595], "z": [2.9718723047396045, 4.626986601170395, 3.476703004034613, 3.4672040607244474, 5.578737152720991, 3.4214028394863214, 4.0090308368449, 3.7637383104060524, 2.349557525752768, 3.470956850478258, 4.0971349789915035, 4.010649682547249, 4.2116949108131045, 2.307078551024986, 3.161326218872709, 4.751808429864546, 3.9287967084851756, 3.1959488317709885, 6.23607288425737, 4.2615915182216675, 2.713323282537255, 4.292578154681832, 2.844514871031341, 4.572575991741108, 3.338761645116886, 2.5221343486885965, 4.523239725950671, 3.9089728525072722, 2.4969994613356263, 4.262475122134853, 4.736293510895493, 4.995316815922539, 4.090944841840222, 3.887473076946366, 3.7999600913422658, 4.4705319262958305, 3.9604299587328113, 2.86612336720494, 3.287402644946414, 3.4559074061309607, 4.141826903176961, 6.150624910807304, 5.578965201156517, 2.5420857657184466, 4.2913277626057695, 4.612870044712003, 4.122010553066656, 1.8355798446281653, 4.23479616846651, 3.9438893568814732, 4.12239823837246, 5.239029671971979, 3.4659946092727427, 3.3879761823230328, 3.7048446208869006, 3.6365009052891684, 4.257475873091362, 5.515010727769352, 5.1829264750272985, 4.635904583940688, 4.612833314032465, 3.0555511591510243, 2.921885668974552, 3.4928585637980905, 3.40180659166772, 3.5791937704479295, 5.143212922047413, 3.855726445606433, 3.6683402564478977, 5.138051755592221, 4.368234261514017, 5.093440681753138, 4.193566774251711, 4.186014617958611, 3.534467678970082, 2.7411614357036864, 2.491546643205818, 4.917782542574547, 4.201515260982836, 2.611786814628723, 5.058463617943905, 3.426455932318882, 3.9989072564989434, 4.529724199656525, 4.705508905298853, 6.007229343501187, 4.370792491768711, 4.7558426228188555, 4.348694890325581, 3.7872802173867335, 3.265502617551942, 2.807967701604327, 4.158489000564888, 3.9048192101846375, 5.581886057641663, 2.8372306145354234, 4.638092970309094, 3.3187472512119864, 4.939640972041694, 3.7450648915923734]}, {"hoverlabel": {"namelength": 0}, "hovertemplate": "blob=2<br>x=%{x}<br>y=%{y}<br>z=%{z}<br>size=%{marker.size}", "legendgroup": "2", "marker": {"color": "#EF553B", "opacity": 0.7, "size": [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1], "sizemode": "area", "sizeref": 0.005917159763313609, "symbol": "circle"}, "mode": "markers", "name": "2", "scene": "scene", "showlegend": true, "type": "scatter3d", "x": [-0.9897619780286748, -1.2120741408069984, -2.4032521827546076, -2.2527371444605153, -2.431340530373503, -2.2699047912833628, -3.2035913917938825, -2.1934004924729567, -2.8750906894783315, -2.5056930803191673, -3.4043136413475787, -1.8324136987205764, -1.7478121415486156, -1.0852771220183284, -1.2707785493558137, -2.116591780141059, -4.235996058568409, -3.58395929940869, -2.5257150088666447, -1.8377130300138762, -2.994096898736455, -2.074755076051193, -3.073538583058573, -3.1618346537891564, -2.1797597563508715, -3.0571368356916, -4.115919088167477, -3.4969085010274243, -2.1742326281239115, -2.5683908613745023, -0.7306167625860291, -2.3933777155991827, -2.8239163176250894, -4.715918179690219, -2.031509136175574, -3.8147057481888993, -2.7370991539503806, -2.890919298914819, -3.6319729772225173, -1.998682621965727, -3.4805302571950567, -3.3011439511262104, -2.0923066482961845, -3.7172530846790615, -2.5208346277661247, -3.03390693508252, -3.394119140588905, -1.8063625664122427, -1.3275748611072655, -2.6415670026672027, -2.572197604622073, -1.7344876868636856, -1.063795231418409, -3.167320233250198, -2.570373736349175, -2.968574411664088, -3.3499575190182425, -2.672777596792102, -1.3659948575729293, -1.748692044287024, -2.828659183960588, -0.39710754948569527, -2.816727166654596, -2.7313818471386786, -2.6397475374636796, -1.6213340889211323, -3.2541879310635493, -1.8778015202279592, -2.392535464005866, -3.0081387292571873, -4.2366620247263835, -2.110730461883024, -1.41849492836342, -1.0972641715200866, -3.0614400564651616, -1.4088068932635016, -3.417881496902619, -3.1290671050144883, -2.990337820714392, -3.3369800690128844, -2.608223073619141, -3.0201705931982685, 0.07811049776054668, -2.210298822932649, -3.8350197280384175, -3.280427140721697, -1.9026126549258477, -2.112609860554112, -0.6940051943475876, -2.060610107440665, -2.290791480288391, -2.342635092620906, -3.2221035471652075, -2.6883112745905327, -1.1074866812879047, -2.2081065672665314, -1.701185078053832, -1.3491275843544273, -2.762627227970103, -2.7649722988426935], "y": [-0.6813929336493254, -0.31363204572347014, -0.3990441705847544, 0.34613995471754516, 0.21367942307525725, -1.042465230808726, 0.3657665688038728, 1.9593221459923045, -1.1389119653683266, -1.0887663012439694, 0.9009050730277505, 1.2967799300946903, -1.1943468728507605, -1.1355945410599704, 1.0027114488189517, -0.32731968501100195, -0.37054603972541045, -0.02000490669461258, -0.7619655258586968, 0.07119460117674126, 1.6302530275013039, -1.2883457983622166, 1.1746204049928997, 0.056532472266996814, 0.9624343558731635, -1.080063664723326, 0.6332456705398456, -0.7593027015986482, -0.08073992076067599, 2.418101043256408, 1.219109504518877, 0.5466297742309121, 0.6094199650299357, -0.2498237771555888, -0.8505996782400487, 0.8160926780900492, 1.4163162978661596, 2.340484829383188, -1.0303751993908292, 0.3324882235084956, 0.15294703100118925, 0.6408837653072252, -1.0481038131847351, -0.07314165226238611, 2.1566253652545484, 1.1265064939700888, -0.10593166230905343, 1.6539145268295938, 0.13840089278789067, 1.9591386114065934, 0.9405143487618775, 0.6158444364438589, 0.7049128633959578, -0.20773558901701272, 1.6505741966202403, 1.0123435840789987, -1.632632799646427, 1.5547675698110115, -1.2016206689790425, 2.6384214043987293, -0.12206153493912547, -0.35812140617968635, 0.21572470726491375, 0.5752286119014901, -0.8914997562320577, 2.383357490848896, 1.7510002740584016, 0.7457367920198833, 0.3309060628536039, 1.144838642693006, 0.6743556071984238, 1.0501674209594438, -0.2696056984835581, -0.005547933690446372, 0.1274282486529769, 0.08763775517827471, -0.7091269679109606, -1.8359039035131826, -1.378833507341935, 0.5827592130820349, 0.5029621751319618, 0.04306691875481489, 1.1529223387136884, -0.043191833486399045, 0.5706516654331778, 0.7073080129904655, -0.23164544883298188, -0.38444606554064453, -0.31995929735504325, -0.8982679065608818, -0.6925281556929417, -0.8605810249971309, 1.5692656273389587, 0.7740425712972355, -1.223803000994192, -1.0041872434738255, 0.7933227546769773, -0.4495121480314679, 0.5261124809796787, -0.41922870724633565], "z": [2.2727980785872215, 3.4961466330172373, 4.093509979810245, 2.3889482997419105, 3.4490930905836223, 1.3056099489237816, 1.2784156908432343, 1.0501104744713032, 5.152587621251403, 1.8293072768790466, 4.05873561441528, 3.6988889837549985, 3.423042412271221, 4.758240317839813, 1.955116139513747, 1.714448728087199, 2.271307525297161, 3.599389876325155, 2.0263677686152786, 4.194076045912101, 3.255295535617181, 2.8020233587146404, 3.3000422214530034, 2.9504483439790787, 5.154827036500835, 3.751738292853567, 2.8756455451532124, 3.055248734860852, 3.843043469350547, 1.9491456249966657, 3.248031167071328, 2.98395549255188, 2.673579069311593, 3.0227632117183405, 2.3387326203470735, 4.940461575755908, 1.8298187634594711, 2.4406956503879904, 2.073188851668772, 2.4805567698703066, 2.0928019342626247, 2.7424425065797644, 1.7230671409432958, 1.0459472153440164, 3.107507832242388, 3.1955655829789498, 2.2881898344762868, 0.7693315923621653, 2.4474728630105633, 2.707731555077157, 1.9216098504220616, 2.584308891031738, 3.9056670454845697, 1.808486364797039, 4.522801531556944, 4.2373482567325285, 2.115686829071518, 2.941319529613906, 5.282504929426926, 1.6380616095539695, 3.174109556347168, 2.1837352158152123, 2.827189870114272, 4.436259666363012, 4.242460680530964, 3.838804961917454, 3.4189835915655853, 2.480024738361849, 4.778592924852491, 3.9915042630476814, 3.246026423977833, 3.5394403042427216, 3.002110268143386, 4.6640467167743775, 2.8192284563349763, 3.3354752472715252, 1.5606847040814074, 3.1409073694343426, 4.824559571092747, 1.6246884417607572, 3.880273431209496, 4.810578866335213, 2.4254746702897556, 4.9284665933178795, 2.6219515980240913, 1.8483736946395184, 4.307340806473374, 3.447073459509714, 2.644425618958747, 3.40599995839433, 3.0990084723423634, 0.6587590303032016, 3.5791379677442583, 4.185251305677171, 2.0954379369203977, 4.45394620789591, 4.689838847544047, 1.8385390049164294, 1.4722044582866933, 3.981277685500497]}, {"hoverlabel": {"namelength": 0}, "hovertemplate": "blob=0<br>x=%{x}<br>y=%{y}<br>z=%{z}<br>size=%{marker.size}", "legendgroup": "0", "marker": {"color": "#00cc96", "opacity": 0.7, "size": [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1], "sizemode": "area", "sizeref": 0.005917159763313609, "symbol": "circle"}, "mode": "markers", "name": "0", "scene": "scene", "showlegend": true, "type": "scatter3d", "x": [0.6465932965476617, 0.5936135372735263, 1.5268141189275617, 3.4848412913744506, 0.2961747591613746, 1.8084043959570448, 1.665040324276494, 1.6441175083235797, 0.39240778788610897, 1.9853268675123652, 0.8485166406035977, 2.0467198620818676, 1.4427214479177768, 1.7276244449933331, 2.189843199333365, 1.1273446611915467, 2.1515175629215975, 2.8334656306049903, 1.3075782564810767, 0.4397906395044189, 3.787192208193076, 1.3753424991179517, 1.6848697595777353, 0.7393116814538598, 3.1696666274652676, 1.4249419080364358, 4.090543664058552, 0.7476806109822385, 0.8899277840026026, 3.0079287400397536, 1.1419738641955641, 1.7032653244865428, 3.643912979661211, 1.6755050910784406, 2.6353636071305426, 1.2795435019331647, 1.5219753671491174, 0.9112584924613449, 2.263449299895353, 0.9864376513689739, 2.7898489525415235, 2.126075144544528, 1.2988933497124413, 2.113625691547038, 0.2899920043066635, 2.7680259552998385, 0.9507719406426969, 2.0680306455894355, 1.9965561921018156, 0.034403995079743455, 2.4704544945377642, 0.9217874347330729, 3.6786339612154695, 0.41602905236874976, 2.3005956964162793, 0.9446565822710525, 2.3826335854804044, 1.50026246037721, 2.1544583525751295, 2.5684086855059265, 2.0129923871419986, 1.1054614114697672, 0.6739963105382802, 1.0132114156333376, 1.8885047801900863, 0.37031609137501253, 0.5123320944703231, 0.1967108126058872, 0.49284982782164977, 1.1222826837612945, 2.030726988007143, 0.13452007603750893, 2.320111735246858, 2.714345581786411, 1.9573723374381458, 2.3010475022660626, 1.5213569352329743, 3.06219418173565, 2.5702022349973164, 2.508322064930002, 1.3357428584002538, -0.8554154624456176, 0.7319219435562221, 2.3882751921582677, 2.6047215044592766, 2.4693803809344366, 0.30407315123039536, 0.2698078860053814, 0.8374387249929991, 1.292527132639691, 1.5006314758829205, 2.2908546097657876, 1.628661821264001, 1.2103589844158225, 0.5722730248501635, 2.197110205681725, 2.2461864899720316, 2.591109303385391, 1.100059679000678, 1.8734403636004109], "y": [8.416913109360989, 6.816770544023932, 9.51774972555253, 8.920532725143733, 7.317125602142291, 8.584471172182571, 8.126159016526378, 9.342216470058377, 8.110067530080283, 7.248835050497859, 7.844806252844688, 9.142263653543491, 8.273061159472462, 6.452069552166211, 9.187850811964642, 7.932324794467012, 8.585764423795624, 8.542533864739424, 8.536658203925272, 7.5718999487178635, 7.9310291841828615, 7.738856663157577, 8.312469575296197, 5.564354493989011, 6.57175703203341, 8.855703255158573, 8.062731859140184, 6.484156740108553, 6.807488595873877, 8.38125792299226, 7.119741623141797, 9.196790146439726, 8.304686916539147, 9.569096699338015, 8.988566946695679, 6.991388459363161, 9.895685225856177, 8.559730946183672, 8.648262020620958, 6.0784447312430565, 7.6318570751126895, 7.3811273982327705, 9.249305278475006, 9.530462410315037, 6.080274802527884, 10.081323671864505, 8.431254397288232, 8.674522123931999, 8.61795165971816, 8.03207592944673, 8.373950690378496, 7.518547041778537, 6.787578051478342, 10.071225397256034, 5.626621179650128, 8.742433213513651, 8.355143572891366, 9.285855363181224, 7.106962532155447, 8.969571137475016, 9.103672327989617, 9.829914578424674, 7.926302994902358, 8.440337733468212, 6.863971297147874, 7.911646860641663, 8.813935227207809, 6.72227579944483, 7.680075947995161, 7.280515352368531, 7.6332556253015, 7.630601330139516, 7.809355045441407, 7.2643469305491015, 7.35724609295888, 7.314298820272366, 7.78203313540259, 8.51764449956274, 7.982992122601492, 8.209929360592481, 8.038708529624406, 8.475591285479478, 7.104918426857796, 7.433605799812183, 5.900093097451294, 7.249311034973902, 9.119583541452963, 7.7868071330426325, 7.647188711076482, 6.727224448988187, 8.115624846047774, 7.224153551507591, 7.813601503727265, 8.894260100949122, 6.135715612784674, 10.120957016072232, 7.245104906668904, 8.206242716641633, 9.359332251484494, 8.634251708063596], "z": [8.149165265118992, 8.969670672895337, 7.606193572090586, 6.87610510107135, 6.661703234079222, 6.820395026724752, 8.662184087720487, 7.733924000312326, 8.285629130815655, 7.490867752432437, 10.428398221671003, 8.188421439287561, 7.352792953681409, 6.3781506121588, 7.537544612994955, 8.161864724410057, 7.697893780570477, 8.726045030085817, 8.449437234563248, 8.875008555462534, 6.539387897258793, 7.617200782103407, 4.3046504734811055, 8.29526573559485, 9.617837439881681, 8.562006751739478, 8.977080710709696, 9.046757931798803, 9.14049403561111, 8.078841219316336, 8.746970515752976, 6.150089332348079, 11.610505342292367, 7.061091921664374, 7.21433088856593, 7.249416255662858, 6.86221004927827, 6.986366628593834, 7.126764493568368, 8.947304521584401, 9.34542980277893, 7.725122382053677, 8.329512679618395, 6.3449403330165985, 8.424047891308655, 8.3969007958088, 8.696330492765252, 9.696605584734694, 8.054032934018855, 8.816465806790246, 8.364772178128959, 7.93955339441825, 8.222099623791015, 9.648770029960987, 6.630367480715821, 8.453226686007449, 7.477122949566424, 6.815798600472647, 8.631811314704997, 8.232116922412189, 7.853168844745359, 8.106041620538178, 5.217448276827248, 6.063087996349602, 8.892523514889382, 7.253772103407549, 7.302791917685545, 5.900586925380162, 6.987383672772534, 7.013235364838552, 7.155166279301629, 7.6909419592067225, 7.166087554242836, 8.863255261687096, 8.014664239360856, 10.138265311190132, 9.51488824049872, 5.611519119363372, 9.26757638624318, 8.093617109541784, 8.446250247847823, 9.232634204206219, 7.653742235901476, 8.719754063554005, 7.601905523939881, 7.077554849632539, 8.69891998803569, 8.856466411776054, 6.953465405954009, 7.297284221698928, 8.641124586284603, 9.034433611732233, 9.05642233513209, 8.738196645710486, 9.66752621686736, 8.527402933344446, 9.218730671759726, 9.702573290184274, 9.242456409434553, 6.423027622225067]}],
                        {"height": 400, "legend": {"itemsizing": "constant", "title": {"text": "blob"}, "tracegroupgap": 0}, "margin": {"b": 0, "l": 0, "r": 0, "t": 0}, "scene": {"domain": {"x": [0.0, 1.0], "y": [0.0, 1.0]}, "xaxis": {"title": {"text": "x"}}, "yaxis": {"title": {"text": "y"}}, "zaxis": {"title": {"text": "z"}}}, "template": {"data": {"bar": [{"error_x": {"color": "#2a3f5f"}, "error_y": {"color": "#2a3f5f"}, "marker": {"line": {"color": "white", "width": 0.5}}, "type": "bar"}], "barpolar": [{"marker": {"line": {"color": "white", "width": 0.5}}, "type": "barpolar"}], "carpet": [{"aaxis": {"endlinecolor": "#2a3f5f", "gridcolor": "#C8D4E3", "linecolor": "#C8D4E3", "minorgridcolor": "#C8D4E3", "startlinecolor": "#2a3f5f"}, "baxis": {"endlinecolor": "#2a3f5f", "gridcolor": "#C8D4E3", "linecolor": "#C8D4E3", "minorgridcolor": "#C8D4E3", "startlinecolor": "#2a3f5f"}, "type": "carpet"}], "choropleth": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "choropleth"}], "contour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "contour"}], "contourcarpet": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "contourcarpet"}], "heatmap": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmap"}], "heatmapgl": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmapgl"}], "histogram": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "histogram"}], "histogram2d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2d"}], "histogram2dcontour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2dcontour"}], "mesh3d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "mesh3d"}], "parcoords": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "parcoords"}], "pie": [{"automargin": true, "type": "pie"}], "scatter": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter"}], "scatter3d": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter3d"}], "scattercarpet": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattercarpet"}], "scattergeo": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergeo"}], "scattergl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergl"}], "scattermapbox": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattermapbox"}], "scatterpolar": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolar"}], "scatterpolargl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolargl"}], "scatterternary": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterternary"}], "surface": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "surface"}], "table": [{"cells": {"fill": {"color": "#EBF0F8"}, "line": {"color": "white"}}, "header": {"fill": {"color": "#C8D4E3"}, "line": {"color": "white"}}, "type": "table"}]}, "layout": {"annotationdefaults": {"arrowcolor": "#2a3f5f", "arrowhead": 0, "arrowwidth": 1}, "coloraxis": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "colorscale": {"diverging": [[0, "#8e0152"], [0.1, "#c51b7d"], [0.2, "#de77ae"], [0.3, "#f1b6da"], [0.4, "#fde0ef"], [0.5, "#f7f7f7"], [0.6, "#e6f5d0"], [0.7, "#b8e186"], [0.8, "#7fbc41"], [0.9, "#4d9221"], [1, "#276419"]], "sequential": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "sequentialminus": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]]}, "colorway": ["#636efa", "#EF553B", "#00cc96", "#ab63fa", "#FFA15A", "#19d3f3", "#FF6692", "#B6E880", "#FF97FF", "#FECB52"], "font": {"color": "#2a3f5f"}, "geo": {"bgcolor": "white", "lakecolor": "white", "landcolor": "white", "showlakes": true, "showland": true, "subunitcolor": "#C8D4E3"}, "hoverlabel": {"align": "left"}, "hovermode": "closest", "mapbox": {"style": "light"}, "paper_bgcolor": "white", "plot_bgcolor": "white", "polar": {"angularaxis": {"gridcolor": "#EBF0F8", "linecolor": "#EBF0F8", "ticks": ""}, "bgcolor": "white", "radialaxis": {"gridcolor": "#EBF0F8", "linecolor": "#EBF0F8", "ticks": ""}}, "scene": {"xaxis": {"backgroundcolor": "white", "gridcolor": "#DFE8F3", "gridwidth": 2, "linecolor": "#EBF0F8", "showbackground": true, "ticks": "", "zerolinecolor": "#EBF0F8"}, "yaxis": {"backgroundcolor": "white", "gridcolor": "#DFE8F3", "gridwidth": 2, "linecolor": "#EBF0F8", "showbackground": true, "ticks": "", "zerolinecolor": "#EBF0F8"}, "zaxis": {"backgroundcolor": "white", "gridcolor": "#DFE8F3", "gridwidth": 2, "linecolor": "#EBF0F8", "showbackground": true, "ticks": "", "zerolinecolor": "#EBF0F8"}}, "shapedefaults": {"line": {"color": "#2a3f5f"}}, "ternary": {"aaxis": {"gridcolor": "#DFE8F3", "linecolor": "#A2B1C6", "ticks": ""}, "baxis": {"gridcolor": "#DFE8F3", "linecolor": "#A2B1C6", "ticks": ""}, "bgcolor": "white", "caxis": {"gridcolor": "#DFE8F3", "linecolor": "#A2B1C6", "ticks": ""}}, "title": {"x": 0.05}, "xaxis": {"automargin": true, "gridcolor": "#EBF0F8", "linecolor": "#EBF0F8", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "#EBF0F8", "zerolinewidth": 2}, "yaxis": {"automargin": true, "gridcolor": "#EBF0F8", "linecolor": "#EBF0F8", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "#EBF0F8", "zerolinewidth": 2}}}, "width": 600},
                        {"responsive": true}
                    ).then(function(){

var gd = document.getElementById('46c6c27a-3afd-4c74-b2c6-caa4f689d6c0');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })
                };
                });
            </script>
        </div>



```python
encoder = keras.models.Sequential([
    keras.layers.Dense(2, input_shape=[3])
])
decoder = keras.models.Sequential([
    keras.layers.Dense(3, input_shape=[2])
])
autoencoder = keras.models.Sequential([encoder, decoder])

autoencoder.compile(
    loss='mse',
    optimizer=keras.optimizers.Nadam(lr=0.1)
)

history = autoencoder.fit(
    X_train, X_train,
    validation_data=(X_test, X_test),
    epochs=20,
    verbose=0
)
plot_train_history(history)
```


![png](homl_ch17_Representation-learning-and-generative-learning-using-autoencoders-and-gans_files/homl_ch17_Representation-learning-and-generative-learning-using-autoencoders-and-gans_5_0.png)



```python
codings = encoder.predict(X)

fig = plt.figure(figsize=(8, 5))
plt.scatter(codings[:, 0], codings[:, 1], c=y)
plt.xlabel('$z_1$', fontsize=16)
plt.ylabel('$z_2$', fontsize=16)
plt.title('AE encoding of 3D-data to 2D', fontsize=20)
plt.show()
```


![png](homl_ch17_Representation-learning-and-generative-learning-using-autoencoders-and-gans_files/homl_ch17_Representation-learning-and-generative-learning-using-autoencoders-and-gans_6_0.png)


## Stacked AE

An AE can learn more complex representations if it has multiple hidden layers; this is called a *stacked AE* (or *deep AE*).
Of course, too many hidden parameters, and the AE will just memorize the input (i.e. map one value per input value and just copy it for the output) without learning any useful representations of the data.
Generally, the architecture is symmetric with respect to the middle hidden layer).

### Implementing a stacked AE using Keras

A stacked AE can be built just like a regular deep MLP.
The same techniques we learned for training a deep ANN apply here, too.

Here is a stacked AE that codes the Fashion MNIST data set.
It is effectively a Dense network with the number of nodes: $784 \to 100 \to 30 \to 100 \to 784$.
Since we use a SELU activation function, LeCun normal initialization could be useful, though improvements would be limited because the model is not very deep.
The author decided to use binary cross-entory for the loss function (instead of MSE) because this speeds up the training; it is effectively modeling the pixel intensity as the probability of being black.


```python
fashion_mnist = keras.datasets.fashion_mnist
(X_train_full, y_train_full), (X_test, y_test) = fashion_mnist.load_data()

X_train = X_train_full[5000:] / 255.0
X_valid = X_train_full[:5000] / 255.0
y_train = y_train_full[5000:]
y_valid = y_train_full[:5000]

mnist_fashion_lbls = ["T-shirt/top", "Trouser", "Pullover", "Dress", "Coat", 
                      "Sandal", "Shirt", "Sneaker", "Bag", "Ankle boot"]
```


```python
stacked_encoder = keras.models.Sequential([
    keras.layers.Flatten(input_shape=[28, 28]),
    keras.layers.Dense(100, activation='selu'),
    keras.layers.Dense(30, activation='selu')
])

stacked_decoder = keras.models.Sequential([
    keras.layers.Dense(100, activation='selu', input_shape=[30]),
    keras.layers.Dense(28 * 28, activation='sigmoid'),
    keras.layers.Reshape([28, 28])
])

stacked_ae = keras.models.Sequential([stacked_encoder, stacked_decoder])

stacked_ae.compile(
    loss='binary_crossentropy',
    optimizer=keras.optimizers.SGD(learning_rate=2.0)
)

history = stacked_ae.fit(
    X_train, X_train,
    validation_data=(X_valid, X_valid),
    epochs=5,
    verbose=1
)
plot_train_history(history)
```

    Train on 55000 samples, validate on 5000 samples
    Epoch 1/5
    55000/55000 [==============================] - 22s 395us/sample - loss: 0.3328 - val_loss: 0.3086
    Epoch 2/5
    55000/55000 [==============================] - 20s 372us/sample - loss: 0.3038 - val_loss: 0.2955
    Epoch 3/5
    55000/55000 [==============================] - 35s 639us/sample - loss: 0.2966 - val_loss: 0.2916
    Epoch 4/5
    55000/55000 [==============================] - 23s 419us/sample - loss: 0.2927 - val_loss: 0.2873
    Epoch 5/5
    55000/55000 [==============================] - 23s 420us/sample - loss: 0.2902 - val_loss: 0.2873



![png](homl_ch17_Representation-learning-and-generative-learning-using-autoencoders-and-gans_files/homl_ch17_Representation-learning-and-generative-learning-using-autoencoders-and-gans_9_1.png)


### Visualizing the reconstructions


```python
def plot_fashion_img(img):
    plt.imshow(img, cmap='binary')
    plt.axis('off')


def plot_fashion_imgs(top_X, bottom_X, n_images):
    fig = plt.figure(figsize=(n_images * 1.5, 3))
    for img_idx in range(n_images):
        plt.subplot(2, n_images, 1 + img_idx)
        plot_fashion_img(top_X[img_idx])
        plt.subplot(2, n_images, 1 + n_images + img_idx)
        plot_fashion_img(bottom_X[img_idx])

def show_reconstructions(mdl, n_images=5):
    reconstructions = mdl.predict(X_valid[:n_images])
    plot_fashion_imgs(X_valid, reconstructions, n_images)
```


```python
show_reconstructions(stacked_ae)
```


![png](homl_ch17_Representation-learning-and-generative-learning-using-autoencoders-and-gans_files/homl_ch17_Representation-learning-and-generative-learning-using-autoencoders-and-gans_12_0.png)


The model is a bit too lossy, still, and the following could be adjusted to improve it:

* make the encoder or decoder deeper
* train the model longer
* make the codings larger

### Visualizing the Fashion MNIST Dataset

Often, an AE is not great for visualization.
Instead, it can be used to reduce the dimensions of a large dataset to lower, but still high, dimensional space, and another reduction techinque can be used for the visualization.
Here is an example of pairing the stacked AE trained above with t-SNE.


```python
from sklearn.manifold import TSNE

X_valid_compressed = stacked_encoder.predict(X_valid)
tsne = TSNE()
X_valid_2d = tsne.fit_transform(X_valid_compressed)

fig = plt.figure(figsize=(10, 10))

for i, lbl in enumerate(mnist_fashion_lbls):
    idx = np.where(y_valid == i)
    plt.scatter(X_valid_2d[idx, 0], X_valid_2d[idx, 1], 
                s=10, cmap='tab10', label=lbl)

plt.legend(loc='center left', bbox_to_anchor=(1, 0.5))
plt.show()
```


![png](homl_ch17_Representation-learning-and-generative-learning-using-autoencoders-and-gans_files/homl_ch17_Representation-learning-and-generative-learning-using-autoencoders-and-gans_14_0.png)


### Unsupervised pretraining using stacked AE

If there is no pretrained model available for transfer learning, and you have a bunch of data with only a small fractino of it labeled, a stacked AE can be used to generate the first few layers of your neural network classifier.
The stacked AE can be fed all of the data to train the encoder.
Then the encoder can be used for the lower layers of the classifier and the training can be completed using the labeled data.

### Tying weights

For a symmetrical AE, it is common to tie the weights of the decoder layers to those of the encoder.
This halves the number of weights in the model, thus speeding up training and limiting the risk of overfitting.

This can be done using a custom layer, `DenseTranspose`.
It takes another dense layer and uses its weights, though it still has its own bias vector.


```python
class DenseTranspose(keras.layers.Layer):
    def __init__(self, dense, activation=None, **kwargs):
        self.dense = dense
        self.activation = keras.activations.get(activation)
        super().__init__(**kwargs)
    
    def build(self, batch_input_shape):
        self.biases = self.add_weight(name='bias', initializer='zeros',
                                      shape=[self.dense.input_shape[-1]])
        super().build(batch_input_shape)
    
    def call(self, inputs):
        z = tf.matmul(inputs, self.dense.weights[0], transpose_b=True)
        return self.activation(z + self.biases)
```


```python
dense_1 = keras.layers.Dense(100, activation='selu')
dense_2 = keras.layers.Dense(30, activation='selu')

tied_encoder = keras.models.Sequential([
    keras.layers.Flatten(input_shape=[28, 28]),
    dense_1, 
    dense_2
])
tied_decoder = keras.models.Sequential([
    DenseTranspose(dense_2, activation='selu'),
    DenseTranspose(dense_1, activation='sigmoid'),
    keras.layers.Reshape([28, 28])
])

tied_ae = keras.models.Sequential([tied_encoder, tied_decoder])
```

### Training one AE at a time

Instead of training the entire stacked AE at once, the layers can be trained individually.
First, the first layer is trained as a separate AE, then the outputs from its encoder is used to train the next layer, and so on.
Then the AE are stacked into a single stacked AE.
This technique is not used too much anymore, though.

## Convolutional AE

For images, convutional layers can be used in the AE.
Generally, the encoder reduces the spatia dimension while increasing the depth and the decoder must regain the spatial dimensionality.
For the decoder, transpose convolutional layers are commonly used.
Here is an example for the Fashion MNIST data.


```python
conv_encoder = keras.models.Sequential([
    keras.layers.Reshape([28, 28, 1], input_shape=[28, 28]),
    keras.layers.Conv2D(16, kernel_size=3, padding='same', activation='selu'),
    keras.layers.MaxPool2D(pool_size=2),
    keras.layers.Conv2D(32, kernel_size=3, padding='same', activation='selu'),
    keras.layers.MaxPool2D(pool_size=2),
    keras.layers.Conv2D(64, kernel_size=3, padding='same', activation='selu'),
    keras.layers.MaxPool2D(pool_size=2),
])

conv_decoder = keras.models.Sequential([
    keras.layers.Conv2DTranspose(32, kernel_size=3, strides=2, padding='valid',
                                 activation='selu', input_shape=[3, 3, 64]),
    keras.layers.Conv2DTranspose(16, kernel_size=3, strides=2, padding='same',
                                 activation='selu'),
    keras.layers.Conv2DTranspose(1, kernel_size=3, strides=2, padding='same',
                                 activation='sigmoid'),
    keras.layers.Reshape([28, 28])
])

conv_ae = keras.models.Sequential([conv_encoder, conv_decoder])

conv_ae.compile(
    loss=keras.losses.BinaryCrossentropy(),
    optimizer=keras.optimizers.SGD(0.1)
)

history = conv_ae.fit(
    X_train, X_train,
    validation_data=(X_valid, X_valid),
    epochs=5,
    verbose=1
)
plot_train_history(history)
```

    Train on 55000 samples, validate on 5000 samples
    Epoch 1/5
    55000/55000 [==============================] - 154s 3ms/sample - loss: 0.3575 - val_loss: 0.3066
    Epoch 2/5
    55000/55000 [==============================] - 153s 3ms/sample - loss: 0.3028 - val_loss: 0.2923
    Epoch 3/5
    55000/55000 [==============================] - 149s 3ms/sample - loss: 0.2930 - val_loss: 0.2859
    Epoch 4/5
    55000/55000 [==============================] - 149s 3ms/sample - loss: 0.2875 - val_loss: 0.2816
    Epoch 5/5
    55000/55000 [==============================] - 151s 3ms/sample - loss: 0.2839 - val_loss: 0.2815



![png](homl_ch17_Representation-learning-and-generative-learning-using-autoencoders-and-gans_files/homl_ch17_Representation-learning-and-generative-learning-using-autoencoders-and-gans_19_1.png)



```python
show_reconstructions(conv_ae)
```


![png](homl_ch17_Representation-learning-and-generative-learning-using-autoencoders-and-gans_files/homl_ch17_Representation-learning-and-generative-learning-using-autoencoders-and-gans_20_0.png)


## Denoising AE

The AE can be forced to learn to denoise by adding random Gaussian noise to the inputs and having it recreate the original image.
Alternatively, dropout can be used, too.


```python
dropout_encoder = keras.models.Sequential([
    keras.layers.Flatten(input_shape=[28, 28]),
    keras.layers.Dropout(0.5),
    keras.layers.Dense(100, activation='selu'),
    keras.layers.Dense(30, activation='selu')
])

dropout_decoder = keras.models.Sequential([
    keras.layers.Dense(100, activation='selu', input_shape=[30]),
    keras.layers.Dense(28 * 28, activation='sigmoid'),
    keras.layers.Reshape([28, 28])
])

dropout_ae = keras.models.Sequential([dropout_encoder, dropout_decoder])
```

## Sparse AE

Forcing sparsity in a larger coding layer using $\ell_1$ regularization will make the AE learn only the most important features.
A simple implementation is to have a larger dense coding layer with a sigmoid activation function and $\ell_1$ regularization added by setting `activity_regularizer=keras.regularizers.l1(1e-3)`.


```python
sparse_l1_encoder = keras.models.Sequential([
    keras.layers.Flatten(input_shape=[28, 28]),
    keras.layers.Dense(100, activation='selu'),
    keras.layers.Dense(300, activation='sigmoid', 
                       activity_regularizer=keras.regularizers.l1(1e-3))
])

sparse_l1_decoder = keras.models.Sequential([
    keras.layers.Dense(100, activation='selu', input_shape=[300]),
    keras.layers.Dense(28 * 28, activation='sigmoid'),
    keras.layers.Reshape([28, 28])
])

sparse_l1_ae = keras.models.Sequential([sparse_l1_encoder, sparse_l1_decoder])
```

Another alternative that generally works better is to measure the actual sparsity and penalize the model when it is different than a target sparsity.
The actual sparsity can be measured as the average activation of each neuron in the coding layer over the whole training batch.
We can then penalize the neurons that are too active or not active enough by adding a *sparsity loss* to the cost function.
A good penalty to use is the Kullback-Leibler (KL) divergence because it has stronger gradients than MSE.

Below is an example of implementing a custom regularizer using KL divergence.
We included a `weight` parameter to control the importance of the KL divergence on the cost function.


```python
K = keras.backend
kld = keras.losses.kullback_leibler_divergence

class KLDivergenceRegularizer(keras.regularizers.Regularizer):
    def __init__(self, weight, target=0.1):
        self.weight = weight
        self.target = target
    def __call__(self, inputs):
        mean_activities = K.mean(inputs, axis=0)
        return self.weight * (
            kld(self.target, mean_activities) + 
            kld(1.0 - self.target, 1 - mean_activities))
```


```python
kld_reg = KLDivergenceRegularizer(weight=0.05, target=0.1)

sparse_kl_encoder = keras.models.Sequential([
    keras.layers.Flatten(input_shape=[28, 28]),
    keras.layers.Dense(100, activation='selu'),
    keras.layers.Dense(300, activation='sigmoid', activity_regularizer=kld_reg)
])

sparse_kl_decoder = keras.models.Sequential([
    keras.layers.Dense(100, activation='selu', input_shape=[300]),
    keras.layers.Dense(28 * 28, activation='sigmoid'),
    keras.layers.Reshape([28, 28])
])

sparse_kl_ae = keras.models.Sequential([sparse_kl_encoder, sparse_kl_decoder])
```

## Variational AE

A *variational AE* is a very popular form of stacked AE.
It is *probabilistic* and *generative*, that is it builds and randomly samples a distribution.
The overall architecture is the same to a normal stacked AE, there is an encoder and a decoder.
However, the low-dimensional coding is different.
Instead of directly making a coding for a given input, the variational AE produces a *mean coding* $\boldsymbol{\mu}$ and a *standard deviation* $\boldsymbol{\sigma}$.
The actual coding is then sampled from that distribution.
The decoder uses that coding like normal.
Interestingly, this means that a variational AE tends to produce codings that look like they come from a simple Gaussian distribution (on the *latent space*).

The cost function has two pieces.
The first is the standard reconstruction loss to push the decoder to recreate the input.
The second, the *latent loss*, pushes the AE to encode a Gaussian distribution using the KL divergence between the target Gaussian distribution and actual distribution of the codings.

The math isn't fully covered here, but the latent loss can be computed using the following equation:

$ \mathscr{L} = -0.5 \sum_{i=1}^{K} 1 + \log{(\sigma_i^2)} - \sigma_i^2 - \mu_i^2 $

where:

* $\mathscr{L}$: latent loss
* $K$: the coding's dimensionality
* $\sigma_i$ and $\mu_i$: the mean and standard deviation of the $i^{th}$ component of the codings.

A common tweak used for performance reasons is to use $\boldsymbol{\gamma} = \log{\boldsymbol{\sigma}^2}$:

$ \mathscr{L} = -0.5 \sum_{i=1}^{K} 1 + \gamma_i - e^{\gamma_i} - \mu_i^2 $

Now we can begin to build a variational AE in Keras for the Fashion MNIST dataset.
First, we must build a custom layer to sample the codings given $\boldsymbol{\mu}$ and $\boldsymbol{\gamma}$.


```python
class SamplingLayer(keras.layers.Layer):
    def call(self, inputs):
        mean, log_var = inputs
        return K.random_normal(tf.shape(log_var)) * K.exp(log_var / 2) + mean
```

Now we can make the AE - we must use the functional API because the AE is not entirely sequential.
We begin by creating the encoder that has three outputs, the mean coding $\boldsymbol{\mu}$, log variation coding $\boldsymbol{\gamma}$, and the sampling from the encoded distributions.


```python
codings_size = 10

inputs = keras.layers.Input(shape=[28, 28])
flatten_layer = keras.layers.Flatten()(inputs)
encoder_layer1 = keras.layers.Dense(150, activation='selu')(flatten_layer)
encoder_layer2 = keras.layers.Dense(100, activation='selu')(encoder_layer1)

codings_mean = keras.layers.Dense(codings_size)(encoder_layer2)
codings_log_var = keras.layers.Dense(codings_size)(encoder_layer2)
codings = SamplingLayer()([codings_mean, codings_log_var])

variational_encoder = keras.Model(
    inputs=[inputs],
    outputs=[codings_mean, codings_log_var, codings]
)
```

Now we can make the decoder (we could have used the Sequential API for this).


```python
decoder_inputs = keras.layers.Input(shape=[codings_size])
decoder_layer1 = keras.layers.Dense(100, activation='selu')(decoder_inputs)
decoder_layer2 = keras.layers.Dense(150, activation='selu')(decoder_layer1)
decoder_layer3 = keras.layers.Dense(28*28, activation='sigmoid')(decoder_layer2)
outputs = keras.layers.Reshape([28, 28])(decoder_layer3)

variational_decoder = keras.Model(
    inputs=[decoder_inputs],
    outputs=[outputs]
)
```

Finally, the variational AE can be assembled from the encoder and decoder.


```python
_, _, codings = variational_encoder(inputs)
reconstructions = variational_decoder(codings)
variational_ae = keras.Model(
    inputs=[inputs],
    outputs=[reconstructions]
)
```

Lastly, we must add the latent loss.
We divide the latent loss by 784 ($28 \times 28$) to scale it down to compare with the reconstruction loss.
This is because, the non-scaled latent loss is the sum of the latent loss of each pixel while the binary cross-entropy computed by Keras is a mean of all the pixels.


```python
latent_loss = -0.5 * K.sum(
    1 + codings_log_var - K.exp(codings_log_var) - K.square(codings_mean),
    axis=-1
)

variational_ae.add_loss(K.mean(latent_loss) / 784.0)

variational_ae.compile(
    loss='binary_crossentropy',
    optimizer=keras.optimizers.RMSprop()
)

history = variational_ae.fit(
    X_train, X_train,
    epochs=10,
    batch_size=128,
    validation_data=(X_valid, X_valid)
)
```

    Train on 55000 samples, validate on 5000 samples
    Epoch 1/10
    55000/55000 [==============================] - 30s 543us/sample - loss: 0.3871 - val_loss: 0.3488
    Epoch 2/10
    55000/55000 [==============================] - 18s 326us/sample - loss: 0.3432 - val_loss: 0.3330
    Epoch 3/10
    55000/55000 [==============================] - 20s 368us/sample - loss: 0.3332 - val_loss: 0.3254
    Epoch 4/10
    55000/55000 [==============================] - 18s 330us/sample - loss: 0.3280 - val_loss: 0.3229
    Epoch 5/10
    55000/55000 [==============================] - 20s 368us/sample - loss: 0.3249 - val_loss: 0.3237
    Epoch 6/10
    55000/55000 [==============================] - 19s 338us/sample - loss: 0.3228 - val_loss: 0.3228
    Epoch 7/10
    55000/55000 [==============================] - 21s 382us/sample - loss: 0.3211 - val_loss: 0.3187
    Epoch 8/10
    55000/55000 [==============================] - 20s 363us/sample - loss: 0.3199 - val_loss: 0.3169
    Epoch 9/10
    55000/55000 [==============================] - 17s 315us/sample - loss: 0.3189 - val_loss: 0.3145
    Epoch 10/10
    55000/55000 [==============================] - 17s 315us/sample - loss: 0.3181 - val_loss: 0.3162



```python
plot_train_history(history)
```


![png](homl_ch17_Representation-learning-and-generative-learning-using-autoencoders-and-gans_files/homl_ch17_Representation-learning-and-generative-learning-using-autoencoders-and-gans_38_0.png)



```python
show_reconstructions(variational_ae)
```


![png](homl_ch17_Representation-learning-and-generative-learning-using-autoencoders-and-gans_files/homl_ch17_Representation-learning-and-generative-learning-using-autoencoders-and-gans_39_0.png)


### Generating Fashion MNIST Images

Now we can use the variational AE to generate *new* images.
We just take 12 random samples from a normal distribution with the correct number of sizes for the decoder.
We then use the decoder to turn these into clothing.


```python
codings = tf.random.normal(shape=[12, codings_size])
images = variational_decoder(codings).numpy()
plot_fashion_imgs(images[:6, :, :], images[6:, :, :], n_images=6)
```


![png](homl_ch17_Representation-learning-and-generative-learning-using-autoencoders-and-gans_files/homl_ch17_Representation-learning-and-generative-learning-using-autoencoders-and-gans_41_0.png)


*Semantic interpolation* is also possible with variational AE.
Instead of interpolating two images at the pixel level, it can interpolate them at the codings level.
Therefore, we can take two codings, create an interpolation of the two, and feed that into the decoder.
The resulting image will have properties of the two original images.

In the following example, we take the same 12 codings generated above, reshape them into a 3 x 4 grid, and then use `tf.image.resize()` to force them into a 5 x 7 grid such that there will be a new row and column between each original row and column.
By default, `resize()` uses bilinear interpolation.
Passing this new grid into the decoder produces the images resulting of the interpolated codings.
The original images (from above) are outlined; the rest were interpolated.


```python
codings_grid = tf.reshape(codings, [1, 3, 4, codings_size])
larger_grid = tf.image.resize(codings_grid, size=[5, 7])

interpolated_codings = tf.reshape(larger_grid, [-1, codings_size])
interpolated_images = variational_decoder(interpolated_codings).numpy()
```


```python
fig, axes = plt.subplots(nrows=5, ncols=7, figsize=(10, 10))
for i, ax in enumerate(axes.flatten()):
    ax.imshow(interpolated_images[i, :, :], cmap='binary')
    
    if i % 2 != 0:
        ax.axis(False)
    else:
        ax.grid(False)
        for side in ['left', 'right', 'top', 'bottom']:
            ax.spines[side].set_color('k')
        ax.set_yticklabels([])
        ax.set_xticklabels([])

plt.show()
```


![png](homl_ch17_Representation-learning-and-generative-learning-using-autoencoders-and-gans_files/homl_ch17_Representation-learning-and-generative-learning-using-autoencoders-and-gans_44_0.png)


## Generative Adversarial Networks

The main idea behind a generative adversarial network (GAN) is to have two neural networks compete against each other: the first tries to generate convincing output and the second tried to discern between the generated and real data.
Thus, a GAN is this composed of two neural networks:

* The *generator* takes a random distribution as input and produces some output. In this way, it is simillar to the decoder of an AE.
* The *discriminator* takes an input and tries to tell if it is fake (i.e. generated by the generator) or real.

There are two phases to training a GAN:

1. The discriminator is trained on equal parts real and fake data.
2. The generator is trained by creating new output and then trying to trick the discriminator.

Importantly, the generator never sees real data - it is trained purely by feedback from the discriminator.

Below, we build a simple GAN for the Fashion MNIST data.
The generator is similar to an AE's decoder and the discriminator is a regular binary classifier.


```python
codings_size = 30

# GAN: generator
generator = keras.models.Sequential([
    keras.layers.Dense(100, activation='selu', input_shape=[codings_size]),
    keras.layers.Dense(150, activation='selu'),
    keras.layers.Dense(28 * 28, activation='sigmoid'),
    keras.layers.Reshape([28, 28])
])

# GAN: discriminator
discriminator = keras.models.Sequential([
    keras.layers.Flatten(input_shape=[28, 28]),
    keras.layers.Dense(150, activation='selu'),
    keras.layers.Dense(100, activation='selu'),
    keras.layers.Dense(1, activation='sigmoid')
])

# complete GAN
gan = keras.models.Sequential([generator, discriminator])
```

The next step is to compile the models.
The discriminator's loss function is naturally binary cross-entropy and the generator does not need a specific loss function since it gets trained with the entire GAN.

Note how the discriminator is compiled, *then* its weights are frozen before compiling the entire GAN.
This is such that the `discriminator` object can be trained separately with the weights not frozen, and the `gan` object can be trained *with* the discriminator's weights frozen.
This works because setting `trainable = False` only has an effect if done *before* compiling.


```python
# NOTE: order matters!
#   compile discriminator -> freeze discriminator -> compile GAN

discriminator.compile(
    loss='binary_crossentropy',
    optimizer=keras.optimizers.RMSprop()
)

discriminator.trainable = False

gan.compile(
    loss='binary_crossentropy',
    optimizer=keras.optimizers.RMSprop()
)
```

Obviously, we must create a custom training loop for the GAN.
First, we must create a `Dataset` object to iterate through the images.


```python
batch_size = 32
X_train = X_train.astype(np.float32)
dataset = tf.data.Dataset.from_tensor_slices(X_train).shuffle(1000)
dataset = dataset.batch(batch_size, drop_remainder=True).prefetch(2)
dataset
```




    <PrefetchDataset shapes: (32, 28, 28), types: tf.float32>




```python
X_train_small = X_train.astype(np.float32)[0:8000]
dataset_small = tf.data.Dataset.from_tensor_slices(X_train_small).shuffle(1000)
dataset_small = dataset_small.batch(batch_size, drop_remainder=True).prefetch(2)
dataset_small
```




    <PrefetchDataset shapes: (32, 28, 28), types: tf.float32>




```python
def train_gan(gan, dataset, batch_size, codings_size, n_epochs=10):
    generator, discriminator = gan.layers
    for epoch in range(n_epochs):
        print(f"epoch {epoch}")
        print("batch:", end=" ")
        for i, X_batch in enumerate(dataset):
            if (i % 10) == 0.0:
                print(i, end=" ")
            # PAHSE 1 - train the discriminator
            # generate new images
            noise = tf.random.normal(shape=[batch_size, codings_size])
            generated_imgs = generator(noise)
            # combine with real and make target vector
            X_fake_and_real = tf.concat([generated_imgs, X_batch], axis=0)
            y1 = tf.constant([[0.0]] * batch_size + [[1.0]] * batch_size)
            # train disciminator
            discriminator.trainable = True
            discriminator.train_on_batch(X_fake_and_real, y1)

            # PHASE 2 - train the generator
            # make noise for input and target vector of all 1's
            noise = tf.random.normal(shape=[batch_size, codings_size])
            y2 = tf.constant([[1.0]] * batch_size)
            # freeze discriminator and train GAN
            discriminator.trainable = False
            gan.train_on_batch(noise, y2)
        print("")
```


```python
train_gan(gan, 
          dataset_small, 
          batch_size, 
          codings_size, 
          n_epochs=2)
```

    epoch 0
    batch: 0 10 20 30 40 50 60 70 80 90 100 110 120 130 140 150 160 170 180 190 200 210 220 230 240 
    epoch 1
    batch: 0 10 20 30 40 50 60 70 80 90 100 110 120 130 140 150 160 170 180 190 200 210 220 230 240 


We can already see that the GAN has learned to generate some shirts.


```python
noise = tf.random.normal(shape=[12, codings_size])
gen_imgs = generator(noise)
plot_fashion_imgs(gen_imgs[:6], gen_imgs[6:], n_images=6)
```


![png](homl_ch17_Representation-learning-and-generative-learning-using-autoencoders-and-gans_files/homl_ch17_Representation-learning-and-generative-learning-using-autoencoders-and-gans_55_0.png)


### The difficulties of training GANs

Ideally, the generator and discriminator of a GAN would reach an equilibrium where the generator is so good that the discriminator is reduced to guessing whether the input image is real or fake.
However, the GAN as we have trained it above is more likely to be rather unstable.
For instance, the generator may tend to learn to create a single class of the possible images, for instance, the GAN trained above seems to only create shirt-looking images.
This will continue until the discriminator gets really good at telling fake from real shirts and then the generator may change to creating another type of image.
Thus, the GAN will bounce around and never learn to make all of the possible clothing types.
Solving this and other problems have been a large portion of the research on GANs, and the next section focuses on some of the better architectures created.

### Deep Convolutional GANs

In 2015, Radford *et al.* proposed the following guidelines for *deep convolutional GANs* (DCGANs):

1. Replace pooling layers with strided convolutions in the discriminator and transposed convolutions in the generator.
2. Use Batch Normalization in both the generator and discriminator except in the generator's output layer and discriminator's input layer.
3. Use ReLU activation in the generator for all layers save for the output layer which should use hyperbolic tangent.
4. Use Leaky ReLU activation in the discriminator for all layers.

Below is the implementation of these guidelines for a model that performs reasonably well with the Fashion MNIST data.


```python
codings_size = 100

dc_generator = keras.models.Sequential([
    keras.layers.Dense(7 * 7 * 128, input_shape=[codings_size]),
    keras.layers.Reshape([7, 7, 128]),
    keras.layers.BatchNormalization(),
    keras.layers.Conv2DTranspose(64, kernel_size=5, strides=2, padding='same',
                                 activation='selu'),
    keras.layers.BatchNormalization(),
    keras.layers.Conv2DTranspose(1, kernel_size=5, strides=2, padding='same', 
                                 activation='tanh')
])

dc_discriminator = keras.models.Sequential([
    keras.layers.Conv2D(64, kernel_size=5, strides=2, padding='same',
                       activation=keras.layers.LeakyReLU(0.2),
                       input_shape=[28, 28, 1]),
    keras.layers.Dropout(0.4),
    keras.layers.Conv2D(128, kernel_size=5, strides=2, padding='same',
                        activation=keras.layers.LeakyReLU(0.2)),
    keras.layers.Dropout(0.4),
    keras.layers.Flatten(),
    keras.layers.Dense(1, activation='sigmoid')
])

dcgan = keras.models.Sequential([dc_generator, dc_discriminator])
```

Overall, the previously stated guidelines were followed rather well, save for the use of the dropout layers in the discriminator instead of Batch Normalization.
The author claims that, in this case, BN was too unstable while using dropout was not.
This emphasizes the need to try many hyperparameter settings for DCGANs.

The GAN's inputs need to be scaled so that there is a third dimension for the channel and the outputs range from -1 to 1 to match the possible outputs of the hyperbolic tangent activation function of the generator's output layer.


```python
X_train_dcgan = X_train_small.reshape(-1, 28, 28, 1) * 2.0 - 1.0
```

The DCGAN can now be trained like before.


```python
X_train_dcgan = X_train_dcgan.astype(np.float32)
dataset_dcgan = tf.data.Dataset.from_tensor_slices(X_train_dcgan).shuffle(1000)
dataset_dcgan = dataset_dcgan.batch(batch_size, drop_remainder=True).prefetch(2)
dataset_dcgan
```




    <PrefetchDataset shapes: (32, 28, 28, 1), types: tf.float32>




```python
# Compile
dc_discriminator.compile(
    loss='binary_crossentropy',
    optimizer=keras.optimizers.RMSprop()
)

dc_discriminator.trainable = False

dcgan.compile(
    loss='binary_crossentropy',
    optimizer=keras.optimizers.RMSprop()
)

# Train
train_gan(dcgan, 
          dataset_dcgan, 
          batch_size, 
          codings_size, 
          n_epochs=2)
```

    epoch 0
    batch: 0 10 20 30 40 50 60 70 80 90 100 110 120 130 140 150 160 170 180 190 200 210 220 230 240 
    epoch 1
    batch: 0 10 20 30 40 50 60 70 80 90 100 110 120 130 140 150 160 170 180 190 200 210 220 230 240 



```python
noise = tf.random.normal(shape=[12, codings_size])
gen_imgs = dc_generator(noise)
gen_imgs = tf.reshape(gen_imgs, [12, 28, 28])
plot_fashion_imgs(gen_imgs[:6], gen_imgs[6:], n_images=6)
```


![png](homl_ch17_Representation-learning-and-generative-learning-using-autoencoders-and-gans_files/homl_ch17_Representation-learning-and-generative-learning-using-autoencoders-and-gans_63_0.png)


It is possible, with a longer-trained DCGAN, to do semantic interpolation.
In fact, the coding levels will take on discrete objects in the images, such as, for images of faces, male and female or glasses on their faces.
However, DCGANs trained with large images run into problems of locally convincing features, but inconsistencies at larger distances (e.g. different sleeve lengths of a shirt).

### Progressive growing of GANs

In 2018, it was found that progressively increasing the output size of the generator (and the input for the discriminator) during training fixed a lot of the larger-scale inconsistencies.
Basically, the generators begins making small images, and as training continues, new convolutional layers are eased in one at a time.

In the same paper, the researchers from Nividia suggested using the following, too:

* *Minibatch standard deviation layer*: Basically, a filter is added near the end of the discriminator that tells it the variation of the images in the batch. If there isn't much variation, this makes it easy for the discriminator to call generated images, helping the generator increase in diversity.
* *equalized learning rate*: This is a method that rescales the weights of a model the ensure the dynamic range of each parameter is the same as the rest. This significantly helps adaptive optimizers (RMSProp, Adam, Nadam, etc.) fit all of the parameters at the same rate during training.
* *pixelwise normalization layer*: After each layer in the generator, add a layer that normalizes each activation based on all of the activations in the same image and same pixel, but across all channels. This technique helps avoid explosions in the activation due to excessive competition between the generator and discriminator.

All of these techniques greatly improved the DCGAN's ability to create large realistic images.
To measure how good an image was though, the authors proposed *StyleGANs*.

### StyleGAN

I will not go into the details of *StyleGANs* here, as they will inevitably be inferior to those in the book, but I will provide a basic overview.

The generator of a normal DCGAN was modified while the discriminator was not.
There were two networks introduced to the generator.
The first, a *mapping network* maps the latent codings of the generator to multiple style vectors, vectors responsible for the style of the generated image at different scales.
The second, the *synthesis network*, begins with a learned constant input layer that passes through multiple convolutional layers, processing style vector input and noise at each layer, to finally produce the image.

The StyleGAN introduced noise to each convolutional layer separately from the generator and before the activation functions were applied to help increase the stochasticity of the generated image.
This was an important insight because it freed the responsibility of being random from the generator.


```python

```