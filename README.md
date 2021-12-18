# Application-Using-Main-Window
## Intoduction :
A main window provides a framework for building an application's user interface. Qt has QMainWindow and its related classes for main window management. QMainWindow has its own layout to which you can add [Qtoolbar](https://doc.qt.io/qt-5/qtoolbar.html)s, [QDockWidgets](https://doc.qt.io/qt-5/qdockwidget.html), a [QMenuBar](https://doc.qt.io/qt-5/qmenubar.html), and a [QStatusBar](https://doc.qt.io/qt-5/qstatusbar.html). The layout has a center area that can be occupied by any kind of widget. You can see an image of the layout below.
![mainwindowlayout](https://user-images.githubusercontent.com/85891554/146638032-65831a11-f57a-4125-8a42-36038506448e.png)  
  _For further details you can check [QMainWindow](https://doc.qt.io/qt-5/qmainwindow.html) documentation ._
  # SpreadSheet
  ## Context
  Our goal consists of adding interactive functionality to the SpreadSheet widgets .
  > _The output should be similar to this window:_
  ![Capture d’écran 2021-12-18 115334](https://user-images.githubusercontent.com/85891554/146638481-85e167c2-c4b8-4848-aab5-6b99e0486bf1.png)
 ## Setup
 Our instructor has provided us with the code of the application but without any functionality here [spreadsheet.zip](https://anassbelcaid.github.io/CS311/homeworks/07_spreadsheet/SpreadSheet_01.zip) .
 ## StatusBar
 The updateStatusBar takes two ints in order to syncrhonize with the selected item from the spreadsheet
 ```cpp
 void updateStatusBar(int, int) 
```
Here is the implementation of this function:
 ```cpp
void SpreadSheet::updateStatusBar(int row, int col)
{
    QString cell{"(%0, %1)"};
   cellLocation->setText(cell.arg(row+1).arg(col+1));

}
```
Which simply change the cellLocation text with the current cell coordinates
We added the makeConnexion() function to connect all the actions. Here is the content of the this function:
 ```cpp
void SpreadSheet::makeConnexions()
{

// --------- Connexion for the  select all action ----/
connect(all, &QAction::triggered,
        spreadsheet, &QTableWidget::selectAll);

// Connection for the  show grid
connect(showGrid, &QAction::triggered,
        spreadsheet, &QTableWidget::setShowGrid);

//Connection for the exit button
connect(exit, &QAction::triggered, this, &SpreadSheet::close);

//connectting the chane of any element in the spreadsheet with the update status bar
connect(spreadsheet, &QTableWidget::cellClicked, this,  &SpreadSheet::updateStatusBar);
}
```
## Go Cell
Now we will add the function for the goCell action. For that, we need to create a Dialog for the user to select a cell.
>_we already did it in  Go [GoDialog](https://anassbelcaid.github.io/CS311/designer/#godialog)_

![godidalog_ui](https://user-images.githubusercontent.com/85891554/146639415-c232636a-3a04-44db-b72f-0507356af9c3.png)

Now we will create connexion between the goCell action:
First we will create the proper slot called goCellSlot to respond to the action trigger.
```cpp
 private slots:
 void goCellSlot();            // Go to a Cell slot
```
connect the action to its proper slot in the makeConnexions function:
```cpp
connect(goCell, &QAction::triggered, this, &SpreadSheet::goCellSlot);
```
Now We will implement the goCellSlot() function:
```cpp
void SpreadSheet::goCellSlot(){
    // creer un dialog
    GoDialog d;
     //executer le dialog
    auto reply=d.exec();

    //verifier si le dialog a ete acceplte
    if(reply == GoDialog::Accepted)
    {


        // extraire le text
        QString cell = d.getCell();
        //extraire la ligne

        int row =  cell[0].toLatin1()- 'A';

        //Extraire la colonne

        cell = cell.remove(0,1);
        int col = cell.toInt()-1;

        // changer la cellule
        statusBar()->showMessage("Changing the current cell" , 2000);
        spreadsheet->setCurrentCell(row, col);


     }

}
```

![Capture d’écran 2021-12-18 123637](https://user-images.githubusercontent.com/85891554/146639590-3bac4022-49de-497a-97a8-3d9d85d6c2b9.png)
![Capture d’écran 2021-12-18 123506](https://user-images.githubusercontent.com/85891554/146639591-d446ac4d-bdb0-4b98-9819-08b10d51235a.png)

## Find Dialog
We will move now for the Find dialog. This dialog prompts the user for a input and seek a cell that contains the entered text.
we need to create a Dialog for the user to Find a cell.


![ ](https://anassbelcaid.github.io/CS311/homeworks/07_spreadsheet/find_dialog_ui.png)

First we will create the proper slot called Find  to respond to the action trigger.
```cpp
 private slots:
 void findSlot();            // Go to a Cell slot
```
connect the action to its proper slot in the makeConnexions function:
```cpp
connect(find, &QAction::triggered, this, &SpreadSheet::findSlot);
```
Now We will implement the goCellSlot() function wich search for a cell that contains the target:
```cpp
void SpreadSheet::findSlot(){
    FindDialog dialog;
     auto reply= dialog.exec();
     auto found= false;

     if(reply == FindDialog::Accepted)
     {
         QString text= dialog.getText();
         for(auto i=0; i<spreadsheet->rowCount(); i++)
         {
             for(auto j=0; j<spreadsheet->columnCount(); j++)
             {
                 if(spreadsheet->item(i,j))
                 {
                   QString searchword = spreadsheet->item(i,j)->text();
                    if(searchword.contains(text))
                    {
                        spreadsheet->setCurrentCell(i,j);
                        found= true;



                    }


                }
}
         }
         if(!found)
               QMessageBox::information(this, "Error", "No such word or expression");


     }
     }
```
![Capture d’écran 2021-12-18 123506](https://user-images.githubusercontent.com/85891554/146652903-7b59c343-1850-4ba7-ade1-e13300ed1de8.png)
![Capture d’écran 2021-12-18 123637](https://user-images.githubusercontent.com/85891554/146652906-ca84dfd0-ff91-4445-bf4f-da904aefe3c6.png)

_if the word we want to find doesn't exist , an error message appears


