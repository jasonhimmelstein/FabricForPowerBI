# Incremental transformation with DataFlow Gen 2 and Pipelines


Dataflows are familiar to Power BI users, as they are based on Power Query. Gen 2 dataflows are particularly interesting as they do not store their data in a proprietary manner, but can output to lakehouses, KQL databases, and more. In this module, we'll use a Gen 2 dataflow to transform a thermostat file before using it to update lakehouse data.

## Transforming the file

In the previous two exercises, we started with clean data. In the real world, these files arrive with some extra formatting that needs to be dealt with before loading.

Navigate to the "Fabric for Power BI Professionals" workspace, and open the "My_Lakehouse" lakehouse. Open the Files node, and add a new folder named "Processing". Navigate to this folder and upload the files "2023 - October.csv"and "2023 - November.csv" that you downloaded in the first exercise.

Navigate back to the **"**Fabric for Power BI Professionals" workspace, select the "New" button from the ribbon, and select "more options". Next, select "Dataflow Gen 2".

Although our data is in a CSV file, it is not stored locally, or in SharePoint, so do NOT select the "Import from a Text/CSV file" option on the main screen. Instead, select the "Get data" button from the ribbon, and then "View more" in the "New sources" section. Select the "Microsoft Fabric" tab, and then Lakehouse.

Select "Lakehouse" from the Connection dropdown

Expand the "Fabric for Power BI Professionals" node on the left, and then the "My_Lakehouse" node, and then the "Files" node. Select the "Processing" folder. Click "Create".

Select the Content column. Change its data type to "Binary" by clicking on the icon on the left side of the header.** **

Click the "Combine" icon on the right side of the column header, or select "Combine - Combine files" from the ribbon. Click "OK" on the dialog box. This will combine all files in the folder. 

Note that we don't have proper column names or data types. We need to perform some Power Query magic to make this work.

Select the "Transform sample file" node in the "Queries" pane.

Click on the table cell in the upper left hand corner of the table. Select "Remove top rows". Enter 5 for the Number of rows, and click "OK".

Click on the same table icon, and select "Use first row as headers".

Select the "Processing" query in the "Queries" pane. Click the "Refresh" button in the ribbon. You will see an error. This is due to the fact that we changed the column names for the sample file, so we need to set their types manually.

Delete the last step in the "Applied steps" window. You will see your data again. Set the columns to their correct types as below. Note that lakehouses do not support the Time data type, so we will use Date/Time instead.

Power Query properly infers the data types of the column so we are ready to proceed. However, lakehouses don't support the "Time" data type, so change it to "Date/Time". 

| Column                     | Type           |
| -------------------------- | -------------- |
| Date                       | Date           |
| Time                       | Date/Time      |
| System Setting             | Text           |
| System Mode                | Text           |
| Calendar Event             | Text           |
| Program Mode               | Text           |
| Cool Set Temp (C)          | Decimal number |
| Heat Set Temp (C)          | Decimal number |
| Current Temp               | Decimal number |
| Current Humidity (%RH)     | Whole number   |
| Outdoor Temp (C)           | Decimal number |
| Wind Speed (km/h)          | Whole number   |
| Cool Stage 1 (sec)         | Whole number   |
| Cool Stage 2 (sec)         | Whole number   |
| Heat Stage 1 (sec)         | Whole number   |
| Heat Stage 2 (sec)         | Whole number   |
| Aux Heat 1 (sec)           | Whole number   |
| Fan (sec)                  | Whole number   |
| DM Offset                  | Decimal number |
| Thermostat Temperature (C) | Decimal number |
| Thermostat Humidity (%RH)  | Whole number   |
| Thermostat Motion          |                |

## Set the destination

Select "Data destination" from the bottom right part of the screen.

Select "Lakehouse".

Leave the "Lakehouse" connection selected, and select "Next".

Select the "Existing table" radio button, then open the same set of nodes as above, but this time, note that you can only select the managed tables in the lakehouse. Select the "thermostat_readings" table. Click the "Next" button.

**IMPORTANT** **- select the "Append" option. We will be adding data, not replacing it.** 

The column names don't match up perfectly (in the earlier exercise they were modified to avoid spaces). select the corresponding column for each destination field.

When ready, click the "Save settings" button on the bottom right.

Select the "Dataflow 1" dropdown in the upper left corner. Change the name to "Get new file".

Click the "Publish" button on the bottom right to publish your dataflow.

The dataflow will now run. When it is complete, the "Refreshed" value will be updated.

Open the "Thermostat report 1" report that you completed in an earlier exercise. Note that October data is now included. This is because Power BI is using DirectLake mode to connect to the data. Drill up on the chart to get a monthly view.

Navigate to the Processing folder in your lakehouse and delete the file 2023 - October.csv.

## Automate file processing with a pipeline

Dataflows can be scheduled, but in our case, there is no point in re-adding the same data over and again. Pipelines give us the ability to not only load data, but also to manipulate files. We'll build a pipeline to retrieve new files, load them to the lakehouse with the dataflow created above, and then archive those same files. 

Navigate to the "Fabric for Power BI Professionals" workspace, and open the "My_Lakehouse" lakehouse. Open the Files node, then add a new folder called "Input" and another folder called "Archives". Navigate to the new "Input" folder, and upload the file "2023 - November.csv" that you downloaded in the first exercise.

Navigate back to the "Fabric for Power BI Professionals" workspace, select the "New" button from the ribbon, and select "more options". Next, select "Data pipeline"**.** Name it "Thermostat file processing"

Select the "Copy data" tile on the main page.

Select the "Workspace" tab, then select "Lakehouse".

Choose "My_Lakehouse" from the Lakehouse dropdown, and click "Next"

Select the "Files" radio button.

Select the "Input" folder, then the "Schema agnostic (binary copy)" checkbox. This allows the pipeline to work with files themselves, not the contents of the files. Click "Next".

Select "Lakehouse" as the data destination, and again select "My_Lakehouse" from the dropdown. Click "Next".

Click Browse for the folder path, and select the "Processing" folder. Click "OK". Select "Preserve hierarchy" for the Copy behavior. Click "Next".

Leave the File format as "Binary" and click "Next".

Deselect "Start data transfer immediately" and click "OK".

Select the "Copy data" action, then select the "Source" tab. Open the Advanced section and select the "Delete files after completion" checkbox.

Select "Dataflow" from the ribbon to add a new dataflow action.

Connect the "On Completion" event from the Copy data activity to the input of the Dataflow activity.

Select the Dataflow activity, and set the name to "Load data".

Click the settings tab, and then select "Get new file" in the Dataflow field.

Add another "Copy data" activity (from the ribbon). Connect the "On success" event from the Dataflow activity to the new Copy data activity.

Select the new "Copy data" activity and configure it identically to the first one, using the "Processing" folder as the source, and the "Archives" folder as the destination.

Save the pipeline with the save icon in the ribbon, and then run the pipeline by selecting the "Run" button in the ribbon.

The pipeline will move the source file into the processing folder, run the dataflow to extract the data into the lakehouse, and then archive the file.

Open the Thermostat report 1 report from the earlier exercise. You should see November's data loaded into it. The November data file should also be in the Archive folder.
