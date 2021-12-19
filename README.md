# Application-Using-Main-Window
## Intoduction :
A main window provides a framework for building an application's user interface. Qt has QMainWindow and its related classes for main window management. QMainWindow has its own layout to which you can add [Qtoolbar](https://doc.qt.io/qt-5/qtoolbar.html)s, [QDockWidgets](https://doc.qt.io/qt-5/qdockwidget.html), a [QMenuBar](https://doc.qt.io/qt-5/qmenubar.html), and a [QStatusBar](https://doc.qt.io/qt-5/qstatusbar.html). The layout has a center area that can be occupied by any kind of widget. You can see an image of the layout below.


![mainwindowlayout](https://user-images.githubusercontent.com/85891554/146638032-65831a11-f57a-4125-8a42-36038506448e.png)  

  _For further details you can check [QMainWindow](https://doc.qt.io/qt-5/qmainwindow.html) documentation ._
  
# Table of Content  
- [SpreadSheet](#SpreadSheet)
  - [Context](#Context)
  - [Setup](#Setup)
  - [StatusBar](#StatusBar)
  - [Go Cell](#Go-Cell)
  - [Find Dialog](#Find-Dialog)
  - [Saving and loading files](#Saving-and-loading-files)
    - [Saving files](#Saving-files)
    - [Save File action](#Save-File-action)
    - [Opening files](#Opening-files)
    - [Open File action](#Open-File-action)
  - [show the recent files](#show-the-recent-files)
  - [New File](#New-File)
  - [Edit Actions](#Edit-Actions)
    - [Copy](#Copy)
    - [Paste](#Paste)
    - [Cut](#Cut)
- [Text Editor](#Text-Editor)
  - [Design the text editor](#Design-the-text-editor)
  - [Adding actions](#Adding-actions)
  - [open file](#open-file)
  - [New file](#New-file)
  - [Save file](#Save-file)
- [Conclusion](#Conclusion)
 
  # SpreadSheet
  ## Context
  
  A spreadsheet is an easy way to store all different kinds of data. These data types can include financial data, customer data and product data. Excel spreadsheets can support more than a million rows and more than 16,000 columns, so you’ll have plenty of space to store a huge amount. That’s what makes them ideal for database creation.
  
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

_if the word we want to find doesn't exist , an error message appears_

![Capture d’écran 2021-12-18 200613](https://user-images.githubusercontent.com/85891554/146653024-e32bb1d5-7305-4bcb-9777-945a7fc228d7.png)
![Capture d’écran 2021-12-18 200628](https://user-images.githubusercontent.com/85891554/146653025-8af0b808-5d4e-492e-a08e-8c2a4ee0828f.png)

# Saving and loading files
## Saving files
We will start by writing a private function saveContent(QSTring filename) to save the content of our spreadsheet in a text file.
- We will add the declaration in the header file
```cpp
 protected:
     void saveContent(QString filename);
```
- For the implementation, we will using two classes:
  - [QFile](https://doc.qt.io/qt-5/qfile.html) which provides an interface to read and write in files.
  - [QTextStream](https://doc.qt.io/qt-5/qtextstream.html) for manipulating objects with a stream such as a file. 
-Here is the complete implementation of this function:
```cpp
void SpreadSheet::saveContent(QString filename) {

    QFile file(filename);

    //2 ouvrir le fichier en mode write
    if(file.open(QIODevice::WriteOnly))
    {
        QTextStream out(&file);

         //parcourir les cellules et sauvegarder leur contenu
        for (int i=0;i<spreadsheet->rowCount() ;i++ ) {
            for(int j=0; j<spreadsheet->columnCount();j++){
                auto cell = spreadsheet->item(i,j);
                if(cell)
                    out << i<< "," << j << "," << cell->text() << endl;
            }

        }
    }


    // fermer le fichier
    file.close();
}
```
## Save File action
Now that we have an operational saveContent function, we could focus on the slot itself.

So first we will create a slot to respond to the action trigger in the header.
```cpp
private slots:
    void saveSlot(); 
```
Now we will add the connexion in the makeConnexion function:
```cpp
 connect(save, &QAction::triggered, this, &SpreadSheet::saveSlot);
```
the implementation of the slot
```cpp
void SpreadSheet::saveSlot()
{
    //Creating a file dialog to choose a file graphically
    auto dialog = new QFileDialog(this);

    //Check if the current file has a name or not
    if(currentFile == "")
    {
       currentFile = dialog->getSaveFileName(this,"choose your file");

       //Update the window title with the file name
       setWindowTitle(currentFile);
    }

   //If we have a name simply save the content
   if( currentFile != "")
   {
           saveContent(currentFile);
   }
}
```
## Opening files
We will start by writing a private function opencontent(QSTring filename) to open the content on our spreadsheet from a text file ,CSV file or the normal form.
- We will add the declaration in the header file
```cpp
 protected:
     void openContent(QString filename);
```

-Here is the complete implementation of this function:
```cpp
void SpreadSheet::openContent(QString filename) {
    // obtenir le fichier
    QFile file(filename);
    if(filename.contains(".")){
        auto l = filename.split(QChar('.'));

        if(l[1] == "csv"){
            if(file.open(QIODevice::ReadOnly)){
                QTextStream in(&file);
                QString line;
                int rowcount1 = 0;
                while(!in.atEnd()){
                    line= in.readLine();
                    auto tokens=line.split(QChar(','));
                    for(auto col1=0;col1<tokens.size();col1++){
                        spreadsheet->setItem(rowcount1,col1,new QTableWidgetItem(tokens[col1]));
                    }
                    rowcount1++;
                }
            }
        }
        else if(l[1] == ""){//tsv format
            if(file.open(QIODevice::ReadOnly)){
                QTextStream in(&file);
                QString line;
                int rowcount1 = 0;
                while(!in.atEnd()){
                    line= in.readLine();
                    auto tokens=line.split(QChar::Tabulation);
                    for(auto col1=0;col1<tokens.size();col1++){
                        spreadsheet->setItem(rowcount1,col1,new QTableWidgetItem(tokens[col1]));
                    }
                    rowcount1++;
                }
            }
        }}else{
            if(file.open(QIODevice::ReadOnly))
               {
                   QString line;
                   QTextStream in(&file);
                   while( !in.atEnd())
                   {
                       line = in.readLine();
                       auto tokens = line.split(QChar(',') );
                       int row = tokens[0].toInt();
                       int col = tokens[1].toInt();
                       spreadsheet->setItem(row, col , new QTableWidgetItem(tokens[2]));
                   }

               }

        }
     file.close();

}

```
>_this function opens a file depending on its extension_
## Open File action
Now that we have an operational openContent function, we could focus on the slot itself.

So first we will create a slot to respond to the action trigger in the header.
```cpp
private slots:
    void loadFileSlot(); 
```
Now we will add the connexion in the makeConnexion function:
```cpp
 connect(open, &QAction::triggered, this, &SpreadSheet::loadFileSlot);
```
the implementation of the slot
```cpp
void SpreadSheet::loadFileSlot(){

        QFileDialog d;
        auto filename = d.getOpenFileName();
        if(filemenu){
            currentFile = new QString(filename);
            setWindowTitle(*currentFile);
           openContent(*currentFile);
        }
}
```
## show the recent files
Now we will write some code to store the five recent files opened or saved on the menu 
- we will add this code on our open and save functions 
```cpp
// i is already declared and initiated with 0
        if(i<5){
        H.append(new QAction(filename,this));
        recent->addAction(H[i]);
        connect(H[i], &QAction::triggered, this, &SpreadSheet::op1);
        i++;   
        }else{
//if we exceed 5 files we will need to clear the last place and move the others
            for (int j=0;j<4 ;j++ ) {
                H[4-j]->setText(H[3-j]->text());
            }
         H[0]->setText(filename);


        }
```
- now we will need to implement the op1() Slot who takes the file name from the QAction
```cpp
void SpreadSheet::op1() {
    // obtenir le fichier
auto b = dynamic_cast<QAction*>(sender());
    QFile file(b->text());

        if(b->text().contains(".")){
            auto l = b->text().split(QChar('.'));
    
            if(l[1] == "csv"){
                if(file.open(QIODevice::ReadOnly)){
                    QTextStream in(&file);
                    QString line;
                    int rowcount1 = 0;
                    while(!in.atEnd()){
                        line= in.readLine();
                        auto tokens=line.split(QChar(','));
                        for(auto col1=0;col1<tokens.size();col1++){
                            spreadsheet->setItem(rowcount1,col1,new QTableWidgetItem(tokens[col1]));
                        }
                        rowcount1++;
                    }
                }
            }
            else if(l[1] == ""){//tsv format
                if(file.open(QIODevice::ReadOnly)){
                    QTextStream in(&file);
                    QString line;
                    int rowcount1 = 0;
                    while(!in.atEnd()){
                        line= in.readLine();
                        auto tokens=line.split(QChar::Tabulation);
                        for(auto col1=0;col1<tokens.size();col1++){
                            spreadsheet->setItem(rowcount1,col1,new QTableWidgetItem(tokens[col1]));
                        }
                        rowcount1++;
                    }
                }
            }}else{
                if(file.open(QIODevice::ReadOnly))
                   {
                       QString line;
                       QTextStream in(&file);
                       while( !in.atEnd())
                       {
                           line = in.readLine();
                           auto tokens = line.split(QChar(',') );
                           int row = tokens[0].toInt();
                           int col = tokens[1].toInt();
                           spreadsheet->setItem(row, col , new QTableWidgetItem(tokens[2]));
                       }
    
                   }
    
            }
     file.close();

}

```


## New File
Now we will implement a NEW Slot that create new file and make the connection with the newfile action
- So first we will create a slot to respond to the action trigger in the header.
```cpp
private slots:
    void New(); 
```
- second , we will make the connection
```cpp
 connect(newfile, &QAction::triggered, this, &SpreadSheet::NEW);
```
- now the implementation of the slot that clear the spreadsheet 
```cpp
void SpreadSheet::NEW(){
    spreadsheet->clear();
    currentFile=nullptr;
    setWindowTitle("untiteld");
}
```
#Edit Actions
## Copy
we will create the copy slot
- So first we will create a slot to respond to the action trigger in the header.
```cpp
private slots:
    void copy(); 
```
- second , we will make the connection
```cpp
connect(Copy, &QAction ::triggered, this, &SpreadSheet::copy);
```
- now the implementation of the Copy slot
```cpp
void SpreadSheet::copy()
{

    QLineEdit* lineEdit = dynamic_cast<QLineEdit*>(focusWidget());
    if (lineEdit)
    {
        // it was a line edit, perform copy
        lineEdit->copy();
    }
}
```
## Paste
we will create the paste slot
- So first we will create a slot to respond to the action trigger in the header.
```cpp
private slots:
    void paste(); 
```
- second , we will make the connection
```cpp
connect(Paste, &QAction ::triggered, this, &SpreadSheet::paste);
```
- now the implementation of the Paste slot
```cpp
void SpreadSheet::paste()
{
   
    QLineEdit*  lineEdit = dynamic_cast <QLineEdit*>(focusWidget());
    if (lineEdit)
    {
        // it was a line edit, perform copy
        lineEdit->paste();
    }
}
```
## Cut
we will create the cut slot
- So first we will create a slot to respond to the action trigger in the header.
```cpp
private slots:
    void cuts(); 
```
- second , we will make the connection
```cpp
connect(cutCell, &QAction ::triggered, this, &SpreadSheet::cuts);
```
- now the implementation of the Paste slot
```cpp
void SpreadSheet::cuts()
{
    QLineEdit* lineEdit = dynamic_cast<QLineEdit*>(focusWidget());
      if (lineEdit)
      {
          // it was a line edit, perform copy
          lineEdit->cut();
      }
}
```
# Text Editor

A text editor is a computer program that lets a user enter, change, store, and usually print text (characters and numbers, each encoded by the computer and its input and output devices, arranged to have meaning to users or to other programs).

## Design the text editor

For this task we are going to use QtDesigner  to implement the toolbar , menubar and the PlainTextEdit 

_this is the final result_

![Capture d’écran 2021-12-19 012608](https://user-images.githubusercontent.com/85891554/146659200-82265144-fb94-4d75-ad99-dc7547d210e0.png)

## Adding actions

some actions are already implemented in the [QPlainTextEdit](https://doc.qt.io/qt-5/qplaintextedit.html) (cut,copy and paste)

```cpp
void textEditor::on_action_Copy_triggered()
{
    ui->plainTextEdit->copy();
    // update the status bar
    ui->statusbar->showMessage("Copying the current selection");

}
```

>_Copy_

```cpp
void textEditor::on_action_Paste_triggered()
{
   ui->plainTextEdit->paste();
   ui->statusbar->showMessage("Pasting the previous copied selection");

}
```

>_Paste_

```cpp
void textEditor::on_action_Cut_triggered()
{
    ui->plainTextEdit->cut();
    ui->statusbar->showMessage("Cutting the current selection");

}
```

>_Cut_

  
##open file
the implementation of open function
```cpp
void textEditor::on_actionOpen_triggered()
{
    QString filename= QFileDialog::getOpenFileName(this,"Open the file");
    QFile file(filename);
    currentFile = filename;
    if(!file.open(QIODevice::ReadOnly | QFile::Text))
        QMessageBox::warning(this,"Warning","Cannot open file :"+file.errorString());
    setWindowTitle(filename);
    QTextStream in(&file);
    QString text = in.readAll();
    ui->plainTextEdit->setPlainText(text);
    file.close();
    ui->statusbar->showMessage("Opening the chosen file...");
}
```
##New file

the implementation to create a new textfile

```cpp
void textEditor::on_actionNew_triggered()
{
    currentFile.clear();
    currentFile=nullptr;
    ui->plainTextEdit->setPlainText(QString());
    ui->statusbar->showMessage("Creating a new file");
}
```

## Save file

the implementation to save a file

```cpp
void textEditor::on_actionSave_triggered()
{
    auto dialog = new QFileDialog(this);
    if(currentFile == "")
    {
      currentFile = dialog->getSaveFileName(this,"choose your file");
       setWindowTitle(currentFile);
    }
   if( currentFile != "")
   {
           saveContent(currentFile);
   }
}
```

now we wre are going tpo add the savContent function

```cpp
void textEditor::saveContent(QString filename)
{
    //Gettign a pointer on the file
    QFile file(filename);

    //Openign the file
    if(file.open(QIODevice::WriteOnly))  //Opening the file in writing mode

    {          auto text = ui->plainTextEdit->toPlainText();

        QTextStream in(&file);


               in << text ;
           }


    file.close();

}
```
# ***THE END***
