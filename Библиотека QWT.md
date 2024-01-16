###### Библиотека виджетов для технических приложений.
###### Данная либа включает в себя виджеты графиков, сладер с градацией, термо-шкалу, прибор стрелочного типа(типа тахометра в авто) и что-то еще...

### Установка на Ubuntu

```bash

sudo apt install libqwt-qt5-dev libqwt-headers

```

### CMake конфигурация (LINUX)

Чтобы включить библиотеку в проект **CMake** придется сделать ручной поиск, для этого в **CMakeLists.txt** проекта нужно добавить такие строчки:

```cmake
find_path(QWT_INCLUDE_DIR NAMES qwt.h PATHS
	/usr/include # < У меня директория с заголовками лежит здесь
	/usr/local/include
	"$ENV{LIB_DIR}/include"
	"$ENV{INCLUDE}"
	PATH_SUFFIXES qwt-qt5 qwt
)

find_library(QWT_LIBRARY NAMES qwt qwt-qt5
	PATHS /usr/lib # < Тут лежит файл libqwt-qt5.so
)
```
  

Затем остается добавить библиотеку к таргету:

```cmake
target_include_directories(polygon_qt PUBLIC
	${QWT_INCLUDE_DIR} # <
)

target_link_libraries(polygon_qt PRIVATE
	Qt5::Widgets
	${QWT_LIBRARY} # <
)
```
### Базовый код для построения графика C++:

```cpp
// Сетка
#include <qwt_plot_grid.h>
#include <qwt_legend.h>
// Кривая
#include <qwt_plot_curve.h>
// Маркеры кривой
#include <qwt_symbol.h>
// Приближение/отдаление
#include <qwt_plot_magnifier.h>
// Возможность перемещения по графику
#include <qwt_plot_panner.h>
// Включить отображение координат курсора и двух перпендикулярных прямых линий в месте его отображения
#include <qwt_plot_picker.h>
#include <qwt_picker_machine.h>
// =============================== >
// Базовые настройки графика

ui->qwtPlot->setTitle("Plotter");
ui->qwtPlot->setCanvasBackground(Qt::white);
ui->qwtPlot->setAxisTitle(QwtPlot::yLeft, "Y");
ui->qwtPlot->setAxisTitle(QwtPlot::xBottom, "X");
ui->qwtPlot->insertLegend(new QwtLegend());

// Сетка
QwtPlotGrid *grid = new QwtPlotGrid();
grid->setMajorPen(QPen(Qt::gray, 2));
grid->attach(ui->qwtPlot);

// Кривая
QwtPlotCurve *curve = new QwtPlotCurve();
curve->setTitle("Demo curve");
curve->setPen(Qt::blue, 6); // Цвет и толщина кривой
curve->setRenderHint(QwtPlotItem::RenderAntialiased, true); // Сглаживание

// Маркеры кривой
QwtSymbol *symbol = new QwtSymbol(QwtSymbol::Ellipse, QBrush(Qt::yellow), QPen(Qt::red, 2), QSize(8, 8));
curve->setSymbol(symbol);

// Добавить точки на ранее созданную кривую
QPolygonF points;
points  << QPointF(1.0, 1.0) // координаты X, Y
		<< QPointF(1.5, 2.0) << QPointF(3.0, 2.0)
		<< QPointF(3.5, 3.0) << QPointF(5.0, 3.0);

curve->setSamples(points); // Ассоциировать набор точек с кривой
curve->attach(ui->qwtPlot); // Отобразить кривую на графике
ui->qwtPlot->replot();

// Задание минимума и максимума осей в ручную
ui->qwtPlot->setAxisScale(QwtPlot::xBottom, 0, 10);

// Включение возможности приближения/удаления графика
QwtPlotMagnifier *magnifier = new QwtPlotMagnifier(ui->qwtPlot->canvas());
magnifier->setMouseButton(Qt::MidButton);

// Включение перемещения по графику
QwtPlotPanner *d_panner = new QwtPlotPanner(ui->qwtPlot->canvas());
d_panner->setMouseButton(Qt::RightButton);

// Включить отображение координат курсора и двух перпендикулярных прямых линий в месте его отображения
QwtPlotPicker *d_picker = new QwtPlotPicker(
	QwtPlot::xBottom, QwtPlot::yLeft,    // Ассоциация с осями
	QwtPlotPicker::CrossRubberBand,      // Стиль перпендикулярных линий
	QwtPicker::ActiveOnly,               // Включение/Отключение
	ui->qwtPlot->canvas()                // Ассоциация с полем
);

// Цвет перпендикулярных линий
d_picker->setRubberBandPen(QColor(Qt::red));

// Цвет координат положения курсора
d_picker->setTrackerPen(QColor(Qt::black));

// Включение вышеописанных функций
d_picker->setStateMachine(new QwtPickerDragPointMachine());
```

[[Qt5]]
