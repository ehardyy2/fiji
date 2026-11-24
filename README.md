//Macro for analyzing punctate particles from confocal files
//Opens specified file type and folder, makes z-projection of specified channel, and analyzing particles after thresholding
//to follow windows opening/closing (slower), comment out first line ("Set batch mode true")
//Creates results folder ("Summary.csv" and folder in directory of thresholded image that was analyzed for particles ("Threshold_MIP")

setBatchMode(true);
run("Clear Results");
//inputFolder = "/Users/jvaughen/Desktop/*lysot/";
inputFolder = "/Users/elizabethhardy/Desktop/Grad/Vaughen/remaining/";

fileList = getFileList(inputFolder);
savePath = inputFolder+"/output/"; //making folder to save output z-projection of threshold
if (!File.exists(savePath)) File.makeDirectory(savePath);
filetype = ".czi";
analysis_channel= 1; //CHANGE THIS TO THE CHANNEL OF INTEREST!! 
//for cells was channel 1. For tissue 20x and 63x, was channel2. 

// Process each .czi file in the folder. FIJI can not open files with spaces 
for (i = 0; i < fileList.length; i++) {
    fileName = fileList[i];
    if (fileName.contains(" ")) {
        newFileName = replace(fileName, " ", "_");
        File.rename(inputFolder + fileName, inputFolder + newFileName);
        fileName = newFileName; // Update to use the renamed file
    }

    if (endsWith(fileName, filetype)) {
        filePath = inputFolder + fileName;
        run("Bio-Formats Importer", "open=" + filePath + " autoscale open_all_series windowless=true quiet=true");

        // Process all images in the stack; if not stack, will take analyzed channel ("analysis_channel")
        while (nImages > 0) {
            mytitle = getTitle();
            baseName = substring(fileName, 0, lastIndexOf(fileName, "."));
      
     //may have to comment out if only one
  //   run("Make Substack...", "channels="+analysis_channel+"-"+analysis_channel);
 
        
        if (nSlices>1) {
         run("8-bit");
            		run("Z Project...", "projection=[Max Intensity]");
           			  }
            saveAs("Tiff", savePath + baseName + "_MIP.tiff");
            
            //make sure "Despeckle" is not removing signal you care about!! 
           run("Despeckle", "stack");
        
        // Thresholding and particle analysis. CHANGE METHOD BASED ON PREFERENCE!!! currently 'intermodes', an old method was mean+2.5SD           
			setAutoThreshold("Intermodes dark stack");
			run("Convert to Mask", "method=Intermodes background=Dark calculate black create");
			saveAs("Tiff", savePath + baseName+ "_mask.tiff");
            run("Analyze Particles...", "size=0-Infinity pixel circularity=0-1 show=Masks summarize in_situ");
            saveAs("Tiff", savePath + baseName + "_COUNTED.tiff");
            
            
             //alternative threshold: stack mean+2.5*Standard-deviation bin for the z-projection
			   /*  
				   getStatistics(area, mean, min, max, std, histogram);
	  			   myThreshold2 = (mean + 2.5 * std);
	               setThreshold(myThreshold2, 255);
	               	run("Convert to Mask", "background=Dark calculate black create");
	               	saveAs("Tiff", savePath + getTitle() + "_mask.tiff");
	               	run("Analyze Particles...", "size=3-Infinity pixel circularity=0.3-1 show=Masks summarize in_situ");
          		*/          
            close();
            close();
            close(mytitle);
        }
    }
}
saveAs("Results", inputFolder+"Summary.csv");
setBatchMode(false);
