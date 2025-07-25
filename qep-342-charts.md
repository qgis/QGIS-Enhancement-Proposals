# QGIS Enhancement: Charts

**Date** 2025-06-20
**Author** Mathieu Pellerin (@nirvn)
**Contact** mathieu@opengis.ch
**Version** QGIS 4.0

## Summary

This QEP aims at addressing a long-standing functionality gap within QGIS: 
out-of-the-box charting capabilities. This crucial means to data visualization 
has long been requested and would be a great addition to the layout designer. 

### Motivations

By creating QGIS-powered charting classes instead of relying on 3rd-party 
libraries (such as D3), we would unlock powerful styling by leveraging our 
pre-existing symbology capabilities. 

Furthermore, the rendered graphs would benefit from a tighter integration 
with map canvas and other layout items through sharing the same symbols, 
drawn using the same text rendering engine, etc. This also means the resulting 
charts retain their vector state when exporting content in supported formats 
(PDFs, SVGs, etc.) and will be CMYK-compatible when running on Qt6 builds.

Finally, developing a set of chart classes in src/core will be beneficial 
to the QGIS server project as well as other sister projects such as QField 
where solutions based on python plugins or QtWebkit-driven renderings simply 
do not work.

### Proposed solution

The proposed solution will add charting classes aimed at behaving similarly
to symbol classes with a render function taking a render context object.
The charting capabilities will then be exposed through the layout designer
via a new layout charts item.

### Qgs{Line,Bar,Pie}ChartPlot classes

A new series of chart classes - QgsLineChartPlot, QgsBarChartPlot, and QgsPieChartPlot -
will be added to render these specific chart items. The classes would be
parented to pre-existing QgsPlot classes and re-implement the render( QgsRenderContext & )
and renderContent( QgsRenderContext &, QRectF ) functions.

Preparatory work will be needed to split Qgs2DPlot into a generic Qgs2DPlot
and a specialized Qgs2DXyPlot so axis-less graphs such as QgsPieChartPlot can
be parented to the generic Qgs2DPlot. The existing classes are marked as non-stable,
permitting this API break.

All three chart style implementations would rely on QgsSymbol classes to
render their data components. E.g., for the bar chart, a QgsFillSymbol would
be used to render the bars. This will ensure that rendered charts within
print and atlas layouts look and feel exactly the same as other items and map
canvases. The chart rendering will inject a couple of variables into a chart
scope to allow for users to leverage data-defined properties.

In addition, a pair of QgsChartPlotRegistry and QgsChartPlotGuiRegistry will be added to
keep track of available chart types. This will also enable python plugins to
extend the current set of chart types. Their functions and members will match
pre-existing QGIS registry classes such as callouts and sensors registries.

### Chart data handling: QgsPlotData

A new QgsPlotData class will be added to cover generic plots/charts series
used to render the charts. The class will also hold the categories used
to render charts when the X axis type is set to represent categories.

```
class CORE_EXPORT QgsPlotData
{
  public:

    QgsPlotData() = default;
    ~QgsPlotData();

    /**
     * Returns the list of series forming the plot data.
     * \note the series' ownership is retained by this object.
     */
    QList<QgsAbstractPlotSeries *> series() const;

    /**
     * Adds a series to the plot data.
     * \note the series' ownership is transferred to this object.
     */
    void addSeries( QgsAbstractPlotSeries *series SIP_TRANSFER );

    /**
     * Clears all series from the plot data.
     */
    void clearSeries();

    QStringList categories() const;
    void setCategories( const QStringList &categories );

  private:

    QList<QgsAbstractPlotSeries *> mSeries;
    QStringList mCategories;
};
```

A QgsAbstractPlotSeries abstract class will be added, holding basic information
such as the series name as well as styling components. A QgsXyPlotSeries
implementation to hold the data used to render line, bar, and pie charts.
The class will return raw values stored in a QList of QPair<double, double>.

```
class CORE_EXPORT QgsAbstractPlotSeries
{
  public:

    QgsAbstractPlotSeries() = default;
    virtual ~QgsAbstractPlotSeries() = default;

    /**
     * Returns the series' name.
     */
    QString name() const;

    /**
     * Sets the series' name.
     */
    void setName( const QString &name );

  private:

    QString mName;
};

class CORE_EXPORT QgsXyPlotSeries : public QgsAbstractPlotSeries
{
  public:

    QgsXyPlotSeries() = default;
    ~QgsXyPlotSeries() = default;

    /**
     * Returns the series' list of XY pairs of double.
     */
    QList<std::pair<double, double>> data() const SIP_SKIP;

    /**
     * Appends a pair for of XY  double values to the series.
     */
    void append( const double &x, const double &y );

    /**
     * Clears the series' data.
     */
    void clear();

  private:

    QList<std::pair<double, double>> mData;
};
```

The Qgs2DPlot class’ render( QgsRenderContext & ) and renderContent( QgsRenderContext &, QRectF )
functions will be tweaked to add a new QgsPlotData pointer. This is possible as
the plot classes are currently marked as unstable.

Finally, a new QgsVectorLayerPlotDataGenerator class will be added that takes
care of generating plot data from vector layers. The class will iterate through
features to prepare the plot data based on a provided set of user configured
series. The class will take care of filtering features as well as aggregating
values. The iteration and data preparation will happen in a non-blocking
manner off the main thread.

### Layout chart item

On the GUI front, this proposal aims at exposing charting through the layout
designer. A new chart item will be added with relevant widgets to configure
the chart’s style as well as a vector layer-driven dataset setup. 

On the rendering front, when rendering a chart in “preview” mode within the
layout designer, the dataset preparation will happen in a non-blocking way off
the main thread, with a busy/synchronization indicator shown as a placeholder
until the dataset is prepared and ready to be rendered. This mimics what is
done with other items such as the map item as well as the elevation profile item.

The chart item will pass on crucial expression scopes - such atlas scope - to
the data generator. This will enable atlas-driven charts within atlas layouts.

## Deliverables

- Implementation of three charting classes covering line, bar, and pie charts
- Implementation of plot data classes as well as a data plot generator for vector layers to dynamically generate data
- Implementation of a layout chart item
- Exposing charting classes to python bindings

## Affected Files

- src/core/qgsplot.*

## Risks

Quite low as the proposal involves adding new set of classes that will not
impact preexisting code for the most part. The tweaking pre-existing plot classes
is allowed as they are marked as unstable, giving us the required freedom to
improve the classes there.

## Performance implications

None

## Further considerations / improvements

While this proposal aims at exposing charting functionalities within the layout
designer, the proposal here could lead to future improvements that are out
of this initial implementation scope. For example, dockable chart panel(s) within
QGIS’ main window could easily be implemented then to offer charting views for
vector layers form the currently opened project. Exposing the charting classes
to the python bindings will also make it easier to prototype additional
improvements on that front.

## Backward compatibility

N/A

