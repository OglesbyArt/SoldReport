package artpricingsystem;
import java.io.*;
import java.util.*;
import java.util.Comparator;
        
public class SoldReport {      
    public static int lineCount;
    public static void printReport ()
    {
        double totalActualSellingPrice=0;
        double totalTargetSellingPrice=0;
        int count=0;
        try
        {
            sortFile();
            SoldPainting tempPainting = new SoldPainting();
            File  paintingFile = new File ("GalleryPaintings.dat");
            
            if (paintingFile.exists())
            {
                RandomAccessFile inFile = new RandomAccessFile (paintingFile, "r");
                while (inFile.getFilePointer()!=inFile.length())
                {
                    if (((lineCount%20) == 0) && (lineCount!=0))
                    {
                        System.out.println("\n\n\n\n");
                        printHeader();
                    }
                    if ((lineCount%20) == 0)
                    {
                        printHeader();
                    }
                    tempPainting.read (inFile);
                    double isSold=tempPainting.getActualSellingPrice();
                    if(isSold!=0)
                    {
                        boolean printed=printRecord(tempPainting);
                        if(printed)
                        {
                            totalActualSellingPrice+=tempPainting.getActualSellingPrice();
                            totalTargetSellingPrice+=tempPainting.getTargetSellingPrice();
                            count++;
                        }
                    }
                }
                inFile.close ();
                computeRatio(totalActualSellingPrice, totalTargetSellingPrice, count);
                
            }
            else
            {
              System.out.println ("\nNo Paintings have been sold in the past year.");
            }
            count=0;
            System.out.println("Press <ENTER> to return to Main Menu");
            UserInterface.pressEnter();

        }
 
        catch (Exception e)
        {
            System.out.println ("***** Error: SoldReport.printReport () *****");
            System.out.println ("\t" + e);
        }
    } 


    public static boolean printRecord (SoldPainting b)
    {
        Date today=new Date();
        Calendar calendar = Calendar.getInstance();
        int lastYear=calendar.get(Calendar.YEAR)-1;
        int thisMonth=calendar.get(Calendar.MONTH)+1;
        int thisDay=calendar.get(Calendar.DAY_OF_MONTH);
        String ytd=thisMonth+"/"+thisDay+"/"+lastYear;
        Date lowerbound = new Date (ytd);
        if(b.dateOfSale.after(lowerbound))
        {    
            System.out.printf ("%-25s%-35s%-25s%-45s%-35s%-30s", b.classification, b.dateOfSale, b.artistLastName, b.titleOfWork, b.targetSellingPrice, b.actualSellingPrice);
            if(b.actualSellingPrice>b.targetSellingPrice)
                System.out.printf ("%-35s\n", true);  
            else System.out.printf ("%-35s\n",false);  
            ++lineCount;
            return true;
        }
        return false;
    }
    
    public static void printHeader()
    {
        System.out.println ("\t\t\t\t\t       Produce Paintings Sold in the Past Year Report       \n");
        System.out.printf ("%-25s%-35s%-25s%-45s%-35s%-30s%-35s\n", "Classification Type", "Sale Date", "Artist Last Name", "Title of Work", "Target Selling Price", "Actual Selling Price", "Actual Price > Target Price Flag");
        System.out.println ("----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------");
        lineCount+=4;
    }
    
    public static void computeRatio(double actual, double target, int count)
    {
        double actualavg=actual/count;
        actualavg=Math.round(actualavg*100);
        actualavg=actualavg/100;
        System.out.print("\n\nThe average Actual Selling Price is: $" + actualavg+ "\t\t");
        double targetavg=target/count;
        targetavg=Math.round(targetavg*100);
        targetavg=targetavg/100;
        System.out.print("The average Target Selling Price is: $"+targetavg + "\t\t");
        double ratio=actualavg/targetavg;
        ratio=Math.round(ratio*100);
        ratio=ratio/100;
        System.out.println("The average ratio of the Actual Selling Price to the Target Selling Price is: " + ratio+"\n");
    }
    
    public static void sortFile() 
    {
       try{
           List<SoldPainting> bp= new ArrayList<SoldPainting>();
           File paintingsFile = new File ("GalleryPaintings.dat");
           File  tempPaintingsFile = new File ("Reports.dat");
           tempPaintingsFile.delete();
           RandomAccessFile oldFile = new RandomAccessFile (paintingsFile, "r");
           //Reads in File to Array 
           while (oldFile.getFilePointer () != oldFile.length ()) 
           {
                SoldPainting tempPainting = new SoldPainting ();
                tempPainting.read(oldFile);
                bp.add(tempPainting);
           } 
           oldFile.close ();
           //Sorts the Array
           Collections.sort(bp,new Comparator<SoldPainting>() {
           @Override
            public int compare(SoldPainting  bp1, SoldPainting  bp2)
            {
                int result=bp1.getClassification().compareTo(bp2.getClassification());
                if(result!=0)
                     return result;
                else if (result==0)    
                    result=bp1.getDateOfSale().compareTo(bp2.getDateOfSale());
                return result;
            }
            });
           //Writes Array to File
           RandomAccessFile newFile = new RandomAccessFile (tempPaintingsFile, "rw");
            for(int j=0;j<bp.size();++j) 
            {
                SoldPainting tempPainting = new SoldPainting ();
                tempPainting=bp.get(j);
                tempPainting.write(newFile);
            }
       }
       catch (Exception e)
        {
            System.out.println ("***** Error: SoldReport.sortFile () *****");
            System.out.println ("\t" + e);
        }
    }
}
