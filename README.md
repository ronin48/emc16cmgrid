//+------------------------------------------------------------------+
//|                       Copyright © 2016, Advisor exposes networks |
//|                                                  jason@em16.com  |
//+------------------------------------------------------------------+
#property copyright "Copyright © 2019, www.emc16.com"
#property link      "www.emc16.com"
#property strict 

//*************

//#property description "Advisor exposes networks and helps to collect profit from any movement"
/*Advisor exposes networks and helps

---------------------

For instance
Lot = 0.02
LotPlus = 0.01
//--------------------***
First lot 0.1    
Second     0.1+0.1=0.2 
Third      0.2+0.1=0.3 
Fourth     0.3+0.1=0.4  
---------------------

========================================

ProfitClose - profit in the deposit currency (for example, we bet $ 100 when the total profit of the grid reaches $ 100, it closes)
TralStart   - profit for the start of the trawl in the deposit currency for example 50 $ i)
TralClose   - closes with lower profits for example the same $ 20
Profit reached 50, trawl turned on, profit continued to grow to 60, then rolled back to 20 and everything closed at $ 40 profit
deposits are removed, an alert pops up with a question to continue working? With the answer OK - is set again.
=========================================

//--------------------------------------------------------------------*/
extern int     MaxBuy            = 100;    //Maximum Buy
extern int     MaxSell           = 100;    //Maximum Sell

extern int     OrdersBuyStop     = 1;     //BuyStop
extern int     OrdersSellStop    = 1;     //SellStop
extern int     OrdersBuyLimit    = 1;     //BuyLimit
extern int     OrdersSellLimit   = 1;     //SellLimit
extern string _="";
extern int     FirstLevelStop    = 50;    //FirstLevel Stop 
extern int     StepBuyStop       = 150;   //BuyStop
extern int     StepSellStop      = 150;   //SellStop
extern int     FirstLevelLimit   = 50;   //FirstLevel Limit 
extern int     StepBuyLimit      = 150;   //BuyLimit
extern int     StepSellLimit     = 150;   //SellLimit

extern string __=""; //__Lot size control
extern double  Lot               = 0.10;  //Lot of the first order.    from the price, then according to the formula
extern double  LotPlus           = 0.01;  //Starting lot addition
extern double  K_Lot             = 1;  //Magnification ratio

extern string ___=""; //___Close order
extern double  equity_percent_from_balances=1.2; //Equity percent
extern double  CloseProfitB      = 0.5;   //Close buy when profit is reached
extern double  CloseProfitS      = 0.5;   //Close sell when profit is reached
extern double  ProfitClose       = 2.0;   //Close all profit is reached
extern double  TralStart         = 2;   //Start of trail when profit is reached
extern double  TralClose         = 3;   //Close everything after rollback to

extern int     slippage          = 5;     //Slippage
extern int     Magic             = 91;    //Magic orders, if -1 then it will pick up everything
extern int     X                 = 0;     //Panel offset Horizontal 
extern int     Y                 = 35;    //Panel offset vertically
extern bool    TradeOFF          = false; //Shutdown order (disabling trading after taking profit)
//extern int     Equity            = 200 
//extern bool    TradeOFF        = true;  //Shutdown order (disabling trading after taking profit)
//--------------------------------------------------------------------////////
double        Equity_Old;
extern bool   CloseAllWithEquityIncrease = false;
extern bool   CloseAllWithEquityDecrease = false;
extern double EquityIncreasePercentage = 10.0;
extern double EquityDecreasePercentage = 10.0;
extern string Note1 = "Equity Increase Decrease Percentage xx%";

extern bool   StopTradeAfterEquityIncreased = false;
extern bool   StopTradeAfterEquityDecreased = false;
bool          StopTradeFlag = false;



//------------------------------------------------////////
double STOPLEVEL;
double StopProfit=0;
string val, GV_kn_BS, GV_kn_SS, GV_kn_BL, GV_kn_SL,GV_kn_TrP,GV_kn_CBA,GV_kn_CSA,GV_kn_CA;
bool D,LANGUAGE;
//-------------------------------------------------------------------- 


int OnInit()

{ 

//----------------------------------
//// Date Expiry 
//-----------------------------****
//////.datetime expiry=D'2009.05.03 00:00';
//-----------------------------****
//////.if(TimeCurrent()>expiry)
//////.   {
//////.   Alert("The EA has expired.Contact vender for new version.e-mail..>> fxalive789@gmail.com");
//////.   }
  
//----------------------------------
////// Account Lock
//-------------------------------
 int UserID= 103020791;
  if ( AccountNumber() == UserID)
  {
           Alert("Ok! This ID can use EA");
      }
 else
  {
          Alert("Sorry,Account ID is invalid .Contact vender for updateEA. e-mail..>> fxalive789@gmail.com");
  }
 
//-----***********-------------------------
//---------------------------------***


   int AN=AccountNumber();
   string FIO=AccountName();

//DrawLABEL("cm","Copyright http://fxalive.net/",50,50,clrGray,ANCHOR_LEFT_LOWER,2);
EditCreate(0,"cm ©11" ,0,1,0,180,20,"www.emc16.com","Arial",8,ALIGN_CENTER,false,CORNER_LEFT_LOWER);

   EventSetTimer(1);
   LANGUAGE=TerminalInfoString(TERMINAL_LANGUAGE)=="English";
   if (IsTesting()) ObjectsDeleteAll(0);
   val = AccountCurrency();
   string GVn=StringConcatenate("cm mg ",AN," ",Symbol());
   if (IsTesting()) GVn=StringConcatenate("Test ",GVn);
   GV_kn_BS=StringConcatenate(GVn," BS");
   GV_kn_SS=StringConcatenate(GVn," SS");
   GV_kn_BL=StringConcatenate(GVn," BL");
   GV_kn_SL=StringConcatenate(GVn," SL");
   GV_kn_TrP=StringConcatenate(GVn," TrP");
   GV_kn_CBA=StringConcatenate(GVn," CBA");
   GV_kn_CSA=StringConcatenate(GVn," CSA");
   GV_kn_CA=StringConcatenate(GVn," CA");
//   GV_kn_EQTY=StringConcatenate(GVn," EQTY");

   RectLabelCreate(0,"cm F"    ,0,229,2,220,404,clrIvory); //ขนาดกว้าง x ยาวเมนูหลัก
   DrawLABEL("cm шт",Text(LANGUAGE,"pcs","pcs"),95,10,clrBlack,ANCHOR_CENTER);
   DrawLABEL("cm шаг",Text(LANGUAGE,"step","step"),40,10,clrBlack,ANCHOR_CENTER);
   int y=40;
   DrawLABEL("cm fs",Text(LANGUAGE,"distance to the first Stop","distance to the first Stop"),220,26,clrBlack,ANCHOR_LEFT);
   ButtonCreate(0,"cm Buy Stop"     ,0,225,y,100,20,"Buy Stop","Arial",8,clrBlack,clrLightGray,clrLightGray,clrNONE,GlobalVariableCheck(GV_kn_BS));y+=22;
   ButtonCreate(0,"cm Sell Stop"    ,0,225,y,100,20,"Sell Stop","Arial",8,clrBlack,clrLightGray,clrLightGray,clrNONE,GlobalVariableCheck(GV_kn_SS));y+=22;
   DrawLABEL("cm fl",Text(LANGUAGE,"distance to the first Limit","distance to the first Limit"),220,y+10,clrBlack,ANCHOR_LEFT);y+=22;
   ButtonCreate(0,"cm Buy Limit"    ,0,225,y,100,20,"Buy Limit","Arial",8,clrBlack,clrLightGray,clrLightGray,clrNONE,GlobalVariableCheck(GV_kn_BL));y+=22;
   ButtonCreate(0,"cm Sell Limit"   ,0,225,y,100,20,"Sell Limit","Arial",8,clrBlack,clrLightGray,clrLightGray,clrNONE,GlobalVariableCheck(GV_kn_SL));

   y=40;
   EditCreate(0,"cm OrdersBuyStop"  ,0,120,y,50,20,IntegerToString(OrdersBuyStop),"Arial",8,ALIGN_CENTER,false);y+=22;
   EditCreate(0,"cm OrdersSellStop" ,0,120,y,50,20,IntegerToString(OrdersSellStop),"Arial",8,ALIGN_CENTER,false);y+=44;
   EditCreate(0,"cm OrdersBuyLimit" ,0,120,y,50,20,IntegerToString(OrdersBuyLimit),"Arial",8,ALIGN_CENTER,false);y+=22;
   EditCreate(0,"cm OrdersSellLimit",0,120,y,50,20,IntegerToString(OrdersSellLimit),"Arial",8,ALIGN_CENTER,false);

   y=18;
   EditCreate(0,"cm FirstLevelStop" ,0,65,y,50,20,IntegerToString(FirstLevelStop),"Arial",8,ALIGN_CENTER,false);y+=22;
   EditCreate(0,"cm StepBuyStop"    ,0,65,y,50,20,IntegerToString(StepBuyStop),"Arial",8,ALIGN_CENTER,false);y+=22;
   EditCreate(0,"cm StepSellStop"   ,0,65,y,50,20,IntegerToString(StepSellStop),"Arial",8,ALIGN_CENTER,false);y+=22;
   EditCreate(0,"cm FirstLevelLimit",0,65,y,50,20,IntegerToString(FirstLevelLimit),"Arial",8,ALIGN_CENTER,false);y+=22;
   EditCreate(0,"cm StepBuyLimit"   ,0,65,y,50,20,IntegerToString(StepBuyLimit),"Arial",8,ALIGN_CENTER,false);y+=22;
   EditCreate(0,"cm StepSellLimit"  ,0,65,y,50,20,IntegerToString(StepSellLimit),"Arial",8,ALIGN_CENTER,false);y+=22;

   DrawLABEL("cm lot"   ,Text(LANGUAGE,"Lot size","Lot size"),220,y+10,clrBlack,ANCHOR_LEFT);
   EditCreate(0,"cm Lot"                           ,0,175,y,50,20,DoubleToString(Lot,2),"Arial",8,ALIGN_CENTER,false);
   EditCreate(0,"cm LotPlus"                       ,0,120,y,50,20,DoubleToString(LotPlus,2),"Arial",8,ALIGN_CENTER,false);
   EditCreate(0,"cm K_Lot"                         ,0,65,y,50,20,DoubleToString(K_Lot,2),"Arial",8,ALIGN_CENTER,false);
   DrawLABEL("cm +" ,"+"                           ,125,y+10,clrBlack,ANCHOR_LEFT);
   DrawLABEL("cm x"                 ,"x"           ,70,y+10,clrBlack,ANCHOR_LEFT);y+=22;

   DrawLABEL("cm prof",Text(LANGUAGE,"Profit taking","Profit taking"),220,y+10,clrBlack,ANCHOR_LEFT);
   DrawLABEL("cm val",val,95,y+10,clrBlack,ANCHOR_CENTER);y+=22;

   ButtonCreate(0,"cm Close Buy"    ,0,225,y,100,20,"Close Buy");
   EditCreate(0,"cm CloseProfitB" ,0,120,y,50,20,DoubleToString(CloseProfitB,2),"Arial",8,ALIGN_CENTER,false);
   ButtonCreate(0,"cm Close Buy A"    ,0,65,y,50,20,"auto","Arial",8,clrBlack,clrLightGray,clrLightGray,clrNONE,GlobalVariableCheck(GV_kn_CBA));y+=22;

   ButtonCreate(0,"cm Close Sell"   ,0,225,y,100,20,"Close Sell");
   EditCreate(0,"cm CloseProfitS" ,0,120,y,50,20,DoubleToString(CloseProfitS,2),"Arial",8,ALIGN_CENTER,false);
   ButtonCreate(0,"cm Close Sell A"   ,0,65,y,50,20,"auto","Arial",8,clrBlack,clrLightGray,clrLightGray,clrNONE,GlobalVariableCheck(GV_kn_CSA));y+=22;

   ButtonCreate(0,"cm Close"        ,0,225,y,100,20,"Close");
   EditCreate(0,"cm ProfitClose"  ,0,120,y,50,20,DoubleToString(ProfitClose,2),"Arial",8,ALIGN_CENTER,false);
   ButtonCreate(0,"cm Close A"        ,0,65,y,50,20,"auto","Arial",8,clrBlack,clrLightGray,clrLightGray,clrNONE,GlobalVariableCheck(GV_kn_CA));y+=22;

   ButtonCreate(0,"cm Tral Profit"  ,0,225,y,100,20,Text(LANGUAGE,"Tral Profit","Tral Profit"),"Arial",8,clrBlack,clrLightGray,clrLightGray,clrNONE,GlobalVariableCheck(GV_kn_TrP));
   EditCreate(0,"cm TralStart"    ,0,120,y,50,20,DoubleToString(TralStart,2),"Arial",8,ALIGN_CENTER,false);
   EditCreate(0,"cm TralClose"    ,0,65,y,50,20,DoubleToString(TralClose,2),"Arial",8,ALIGN_CENTER,false);y+=22;

//Shutdown    
   ButtonCreate(0,"cm off"  ,0,225,y,210,20,Text(LANGUAGE,"Shutdown after taking profit","Shutdown after taking profit"),"Arial",8,clrBlack,clrLightGray,clrLightGray,clrNONE,TradeOFF);

//Eq
//     ButtonCreate(0,"cm Equity" ,0,225,y,210,25,("Shutdown after Equity is reached %"),"Arial",8,clrBlack,clrLightGray,clrLightGray,clrNONE,TradeOFF);
//   ButtonCreate(0,"cm Equity" ,0,225,y,210,25,("Shutdown after Equity is reached %"),"Arial",8,clrBlack,clrLightGray,clrLightGray,clrNONE,TradeOFF);
//   EditCreate(0,"cm ProfitClose"  ,0,120,y,50,20,DoubleToString(ProfitClose,2),"Arial",8,ALIGN_CENTER,false);
//   ButtonCreate(0,"cm Close A"        ,0,65,y,50,20,"auto","Arial",8,clrBlack,clrLightGray,clrLightGray,clrNONE,GlobalVariableCheck(GV_kn_EQTY));y+=22;
//..

   ButtonCreate(0,"cm Clear"  ,0,75-X,25-Y,70,20,Text(LANGUAGE,"Clear","Clear") ,"Times New Roman",8, clrBlack,clrGray,clrLightGray,clrNONE,false,CORNER_RIGHT_LOWER);
   return(INIT_SUCCEEDED);
     
}

//-------------------------------------------------------------------
void OnTick() {OnTimer();}
void OnTimer()

{
   if (!IsTradeAllowed()) return;
   STOPLEVEL=MarketInfo(Symbol(),MODE_STOPLEVEL);

   OrdersBuyStop=(int)StringToInteger(ObjectGetString(0,"cm OrdersBuyStop",OBJPROP_TEXT));
   OrdersSellStop=(int)StringToInteger(ObjectGetString(0,"cm OrdersSellStop",OBJPROP_TEXT));
   OrdersBuyLimit=(int)StringToInteger(ObjectGetString(0,"cm OrdersBuyLimit",OBJPROP_TEXT));
   OrdersSellLimit=(int)StringToInteger(ObjectGetString(0,"cm OrdersSellLimit",OBJPROP_TEXT));

   FirstLevelStop=(int)StringToInteger(ObjectGetString(0,"cm FirstLevelStop",OBJPROP_TEXT));
   FirstLevelLimit=(int)StringToInteger(ObjectGetString(0,"cm FirstLevelLimit",OBJPROP_TEXT));
   StepBuyStop=(int)StringToInteger(ObjectGetString(0,"cm StepBuyStop",OBJPROP_TEXT));
   StepSellStop=(int)StringToInteger(ObjectGetString(0,"cm StepSellStop",OBJPROP_TEXT));
   StepBuyLimit=(int)StringToInteger(ObjectGetString(0,"cm StepBuyLimit",OBJPROP_TEXT));
   StepSellLimit=(int)StringToInteger(ObjectGetString(0,"cm StepSellLimit",OBJPROP_TEXT));

   CloseProfitB=StringToDouble(ObjectGetString(0,"cm CloseProfitB",OBJPROP_TEXT));
   CloseProfitS=StringToDouble(ObjectGetString(0,"cm CloseProfitS",OBJPROP_TEXT));
   ProfitClose=StringToDouble(ObjectGetString(0,"cm ProfitClose",OBJPROP_TEXT));
   TralStart=StringToDouble(ObjectGetString(0,"cm TralStart",OBJPROP_TEXT));
   TralClose=StringToDouble(ObjectGetString(0,"cm TralClose",OBJPROP_TEXT));
 //---  EquityClose=StringToDouble(ObjectGetString(0,"cm Equity",OBJPRO_TEXT)); 
   { Comment("     ",AccountName(),"              ACCOUNT NO.  ",AccountNumber(),"           FREE MARGIN  ",AccountFreeMargin(),"          EQUITY  ",AccountEquity(),"            BALANCE  ",AccountBalance());
// DrawLABEL("ecm","Copyright http://fxalive.net/",50,80,clrYellow,ANCHOR_LEFT_LOWER,2);
}
   Lot=StringToDouble(ObjectGetString(0,"cm Lot",OBJPROP_TEXT));
   LotPlus=StringToDouble(ObjectGetString(0,"cm LotPlus",OBJPROP_TEXT));
   K_Lot=StringToDouble(ObjectGetString(0,"cm K_Lot",OBJPROP_TEXT));
   TradeOFF=ObjectGetInteger(0,"cm off",OBJPROP_STATE);
   //---
   double OL,OOP,Profit=0,ProfitB=0,ProfitS=0;
   double OOPBS=0,OOPSS=0,OOPBL=0,OOPSL=0,OLBS=0,OLSS=0,OLBL=0,OLSL=0;
   int OTBS=0,OTSS=0,OTBL=0,OTSL=0;
   int Ticket,i,b=0,s=0,bs=0,ss=0,bl=0,sl=0,tip;
   for (i=0; i<OrdersTotal(); i++)
   {    
      if (OrderSelect(i,SELECT_BY_POS,MODE_TRADES))
      { 
         if (OrderSymbol()==Symbol() && (Magic==-1 || Magic==OrderMagicNumber()))
         { 
            tip = OrderType(); 
            OOP = NormalizeDouble(OrderOpenPrice(),Digits);
            Profit=OrderProfit()+OrderSwap()+OrderCommission();
            Ticket=OrderTicket();
            OL=OrderLots();
            if (tip==OP_BUY)             
            {  
               ProfitB+=Profit;
               b++; 
            }                                         
            if (tip==OP_SELL)        
            {
               ProfitS+=Profit;
               s++;
            } 
            if (tip==OP_BUYSTOP)             
            {  
               if (OOPBS<OOP) {OOPBS=OOP;OTBS=Ticket;OLBS=OL;}
               bs++; 
            }                                         
            if (tip==OP_SELLSTOP)        
            {
               if (OOPSS>OOP || OOPSS==0) {OOPSS=OOP;OTSS=Ticket;OLSS=OL;}
               ss++;
            } 
            if (tip==OP_BUYLIMIT)             
            {  
               if (OOPBL>OOP || OOPBL==0) {OOPBL=OOP;OTBL=Ticket;OLBL=OL;}
               bl++; 
            }                                         
            if (tip==OP_SELLLIMIT)        
            {
               if (OOPSL<OOP) {OOPSL=OOP;OTSL=Ticket;OLSL=OL;}
               sl++;
            } 
         }
      }
   } 
   Profit = ProfitB + ProfitS;
   ObjectSetString(0,"cm Close Buy",OBJPROP_TEXT,StringConcatenate("CloseBuy ",DoubleToStr(ProfitB,2)));   
   ObjectSetString(0,"cm Close Sell",OBJPROP_TEXT,StringConcatenate("CloseSell ",DoubleToStr(ProfitS,2)));   
   ObjectSetString(0,"cm Close",OBJPROP_TEXT,StringConcatenate("Close ",DoubleToStr(Profit,2)));   
   ObjectSetInteger(0,"cm Close Buy",OBJPROP_COLOR,Color(ProfitB));
   ObjectSetInteger(0,"cm Close Sell",OBJPROP_COLOR,Color(ProfitS));
   ObjectSetInteger(0,"cm Close",OBJPROP_COLOR,Color(Profit));
   
   ObjectSetString(0,"cm Buy Stop",OBJPROP_TEXT,StringConcatenate("BuyStop ",bs));   
   ObjectSetString(0,"cm Sell Stop",OBJPROP_TEXT,StringConcatenate("SellStop ",ss));   
   ObjectSetString(0,"cm Buy Limit",OBJPROP_TEXT,StringConcatenate("BuyLimit ",bl));   
   ObjectSetString(0,"cm Sell Limit",OBJPROP_TEXT,StringConcatenate("SellLimit ",sl));   
   //---
   if (ObjectGetInteger(0,"cm Clear",OBJPROP_STATE))
   {
      ObjectsDeleteAll(0,OBJ_TEXT);
      ObjectsDeleteAll(0,"#");
      ObjectsDeleteAll(0,OBJ_ARROW);
      ObjectsDeleteAll(0,OBJ_TREND);
      ObjectSetInteger(0,"cm Clear",OBJPROP_STATE,false);
   }
   //-----------------------------------
   //----- Closing on profit  --------
   //-----------------------------------
   //--- buy
   if (ObjectGetInteger(0,"cm Close Buy A",OBJPROP_STATE)) 
   {  
      GlobalVariableSet(GV_kn_CBA,1);
      ObjectSetInteger(0,"cm CloseProfitB",OBJPROP_COLOR,clrRed);
      if (b!=0 && ProfitB>=CloseProfitB) ObjectSetInteger(0,"cm Close Buy",OBJPROP_STATE,true);
   }
   else {ObjectSetInteger(0,"cm CloseProfitB",OBJPROP_COLOR,clrLightGray);GlobalVariableDel(GV_kn_CBA);}
   if (ObjectGetInteger(0,"cm Close Buy",OBJPROP_STATE))
   {
      if (b!=0) if (!CloseAll(OP_BUY)) Print("Error OrderSend ",GetLastError());
      else 
      {
         if (TradeOFF) ObjectSetInteger(0,"cm Buy Stop",OBJPROP_STATE,false);
         if (TradeOFF) ObjectSetInteger(0,"cm Buy Limit",OBJPROP_STATE,false);
         ObjectSetInteger(0,"cm Close Buy",OBJPROP_STATE,false);
      }
   }
   //--- sell 
   if (ObjectGetInteger(0,"cm Close Sell A",OBJPROP_STATE)) 
   {
      GlobalVariableSet(GV_kn_CSA,1);
      ObjectSetInteger(0,"cm CloseProfitS",OBJPROP_COLOR,clrRed);
      if (s!=0 && ProfitS>=CloseProfitS) ObjectSetInteger(0,"cm Close Sell",OBJPROP_STATE,true); 
   }
   else {ObjectSetInteger(0,"cm CloseProfitS",OBJPROP_COLOR,clrLightGray);GlobalVariableDel(GV_kn_CSA);}
   if (ObjectGetInteger(0,"cm Close Sell",OBJPROP_STATE))
   {
      if (s!=0) if (!CloseAll(OP_SELL)) Print("Error OrderSend ",GetLastError());
      else 
      {
         if (TradeOFF) ObjectSetInteger(0,"cm Sell Stop",OBJPROP_STATE,false);
         if (TradeOFF) ObjectSetInteger(0,"cm Sell Limit",OBJPROP_STATE,false);
         ObjectSetInteger(0,"cm Close Sell",OBJPROP_STATE,false);
      }
   }
   //--- all
   if (ObjectGetInteger(0,"cm Close A",OBJPROP_STATE)) 
   {
      GlobalVariableSet(GV_kn_CA,1);
      ObjectSetInteger(0,"cm ProfitClose",OBJPROP_COLOR,clrRed);
      if (b+s!=0 && Profit>=ProfitClose) ObjectSetInteger(0,"cm Close",OBJPROP_STATE,true);
   }
   else {ObjectSetInteger(0,"cm ProfitClose",OBJPROP_COLOR,clrLightGray);GlobalVariableDel(GV_kn_CA);}
   if (ObjectGetInteger(0,"cm Close",OBJPROP_STATE))
   {
      if (s+b!=0) if (!CloseByOrders()) Print("Error OrderSend ",GetLastError());
      else 
      {
         if (TradeOFF) ObjectSetInteger(0,"cm Buy Stop",OBJPROP_STATE,false);
         if (TradeOFF) ObjectSetInteger(0,"cm Buy Limit",OBJPROP_STATE,false);
         if (TradeOFF) ObjectSetInteger(0,"cm Sell Stop",OBJPROP_STATE,false);
         if (TradeOFF) ObjectSetInteger(0,"cm Sell Limit",OBJPROP_STATE,false);
         ObjectSetInteger(0,"cm Close",OBJPROP_STATE,false);
      }
   }
   //------------------------------
   //--- Opening orders ---------
   //------------------------------
   double lots,price;
   if (ObjectGetInteger(0,"cm Buy Stop",OBJPROP_STATE))
   {
      if (bs<OrdersBuyStop)
      {
         if (bs==0) 
         {
            lots = Lot; price = NormalizeDouble(Ask+FirstLevelStop*Point,Digits);
            if (FirstLevelStop==0 && b==0)
            {
               if (OrderSend(Symbol(),OP_BUY, lots,price,slippage,0,0,"BuyStop#99",Magic,0,clrNONE)==-1) 
                  Print("First position opening error Buy<<",GetLastError(),">>  ");
               else lots = 0;
            }
         }
         else       {lots = OLBS*K_Lot+LotPlus; price = NormalizeDouble(OOPBS+StepBuyStop*Point,Digits);}
         if (lots!=0 && b+bl+bs<MaxBuy)
            if (OrderSend(Symbol(),OP_BUYSTOP, lots,price,slippage,0,0,"BuyStop#99",Magic,0,clrNONE)==-1) 
               Print("Order opening error BS<<",GetLastError(),">>  ");
      }
      GlobalVariableSet(GV_kn_BS,1);
   }
   else
   {
      GlobalVariableDel(GV_kn_BS);
      if (bs>0) if (!OrderDelete(OTBS)) Print("Error <<",GetLastError(),">>  ");
   }
   //---
   if (ObjectGetInteger(0,"cm Sell Stop",OBJPROP_STATE))
   {
      if (ss<OrdersSellStop)
      {
         if (ss==0) 
         {
            lots = Lot; price = NormalizeDouble(Bid-FirstLevelStop*Point,Digits);
            if (FirstLevelStop==0 && s==0)
            {
               if (OrderSend(Symbol(),OP_SELL, lots,price,slippage,0,0,"SellStop#99",Magic,0,clrNONE)==-1) 
                  Print("First position opening error Sell<<",GetLastError(),">>  ");
               else lots = 0;
            }
         }
         else {lots = OLSS*K_Lot+LotPlus; price = NormalizeDouble(OOPSS-StepSellStop*Point,Digits);}
         if (lots!=0 && s+sl+ss<MaxSell)
            if (OrderSend(Symbol(),OP_SELLSTOP, lots,price,slippage,0,0,"SellStop#99",Magic,0,clrNONE)==-1) 
               Print("Order opening error SS<<",GetLastError(),">>  ");
      }
      GlobalVariableSet(GV_kn_SS,1);
   }
   else
   {
      GlobalVariableDel(GV_kn_SS);
      if (ss>0) if (!OrderDelete(OTSS)) Print("Error <<",GetLastError(),">>  ");
   }
   //---
   if (ObjectGetInteger(0,"cm Buy Limit",OBJPROP_STATE))
   {
      if (bl<OrdersBuyLimit)
      {
         if (bl==0)    
         {
            lots = Lot; price = NormalizeDouble(Bid-FirstLevelLimit*Point,Digits);
            if (FirstLevelLimit==0 && b==0)
            {
               if (OrderSend(Symbol(),OP_BUY, lots,NormalizeDouble(Ask,Digits),slippage,0,0,"BuyLimit#99",Magic,0,clrNONE)==-1) 
                  Print("First position opening error Buy<<",GetLastError(),">>  ");
               else lots = 0;
            }
         }
         else {lots = OLBL*K_Lot+LotPlus; price = NormalizeDouble(OOPBL-StepBuyLimit*Point,Digits);}
         if (lots!=0 && b+bl+bs<MaxBuy)
            if (OrderSend(Symbol(),OP_BUYLIMIT, lots,price,slippage,0,0,"BuyLimit#99",Magic,0,clrNONE)==-1) 
               Print("Order opening error BL<<",GetLastError(),">>  ");
      }
      GlobalVariableSet(GV_kn_BL,1);
   }
   else
   {
      GlobalVariableDel(GV_kn_BL);
      if (bl>0) if (!OrderDelete(OTBL)) Print("Error <<",GetLastError(),">>  ");
   }
   //---
   if (ObjectGetInteger(0,"cm Sell Limit",OBJPROP_STATE))
   {
      if (sl<OrdersSellLimit)
      {
         if (sl==0)    
         {
            lots = Lot; price = NormalizeDouble(Ask+FirstLevelLimit*Point,Digits);
            if (FirstLevelLimit==0 && s==0)
            {
               if (OrderSend(Symbol(),OP_SELL, lots,NormalizeDouble(Bid,Digits),slippage,0,0,"SellLimit#99",Magic,0,clrNONE)==-1) 
                  Print("First position opening error Sell<<",GetLastError(),">>  ");
               else lots = 0;
            }
         }
         else {lots = OLSL*K_Lot+LotPlus; price = NormalizeDouble(OOPSL+StepSellLimit*Point,Digits);}
         if (lots!=0 && s+sl+ss<MaxSell)
            if (OrderSend(Symbol(),OP_SELLLIMIT, lots,price,slippage,0,0,"SellLimit#99",Magic,0,clrNONE)==-1) 
               Print("Order opening error SELLLIMIT <<",GetLastError(),">>  ");
      }
      GlobalVariableSet(GV_kn_SL,1);
   }
   else
   {
      GlobalVariableDel(GV_kn_SL);
      if (sl>0) if (!OrderDelete(OTSL)) Print("Error <<",GetLastError(),">>  ");
   }
   //---------------------------------
   //--------- Profit trawl ----------
   //---------------------------------
   if (ObjectGetInteger(0,"cm Tral Profit",OBJPROP_STATE)) //trawl
   {
      ObjectSetInteger(0,"cm TralStart",OBJPROP_COLOR,clrRed); 
      ObjectSetInteger(0,"cm TralClose",OBJPROP_COLOR,clrRed); 
      GlobalVariableSet(GV_kn_TrP,1);
      if (TralClose>TralStart) TralClose=TralStart;
      if (Profit>=TralStart && StopProfit==0) StopProfit=Profit;
      if (Profit>=StopProfit  && StopProfit!=0) StopProfit=Profit;
      if (StopProfit!=0) ObjectSetString(0,"cm Tral Profit",OBJPROP_TEXT,StringConcatenate("Tral ",DoubleToStr(StopProfit-TralClose,2)));
      else ObjectSetString(0,"cm Tral Profit",OBJPROP_TEXT,Text(LANGUAGE,"Tral profit","Tral profit"));
      if (Profit<=StopProfit-TralClose && StopProfit!=0) 
      {
         if (CloseByOrders())
         {
            if (TradeOFF) ObjectSetInteger(0,"cm Buy Stop",OBJPROP_STATE,false);
            if (TradeOFF) ObjectSetInteger(0,"cm Buy Limit",OBJPROP_STATE,false);
            if (TradeOFF) ObjectSetInteger(0,"cm Sell Stop",OBJPROP_STATE,false);
            if (TradeOFF) ObjectSetInteger(0,"cm Sell Limit",OBJPROP_STATE,false);
         }
         StopProfit=0;
         drawtext(StringConcatenate("rl ",TimeToStr(TimeCurrent(),TIME_DATE|TIME_MINUTES)), Time[0], Bid, StringConcatenate("Close all ",DoubleToStr(Profit,2)) ,clrYellow);
         return;
      }
   } 
   else 
   {
      GlobalVariableDel(GV_kn_TrP);
      StopProfit=0;ObjectSetInteger(0,"cm TralStart",OBJPROP_COLOR,clrLightGray);
      ObjectSetInteger(0,"cm TralClose",OBJPROP_COLOR,clrLightGray);
      ObjectSetString(0,"cm Tral Profit",OBJPROP_TEXT,Text(LANGUAGE,"Tral profit","Tral profit"));
   }
}
//--------------------------------------------------------------------
color Color(double P)
{
   if (P>0) return(clrGreen);
   if (P<0) return(clrRed);
   return(clrGray);
}
//------------------------------------------------------------------
void DrawLABEL(string name, string Name, int x, int y, color clr,ENUM_ANCHOR_POINT align=ANCHOR_RIGHT,int CORNER=1)
{
   if (ObjectFind(name)==-1)
   {
      ObjectCreate(name, OBJ_LABEL, 0, 0, 0);
      ObjectSet(name, OBJPROP_CORNER, CORNER);
      ObjectSet(name, OBJPROP_XDISTANCE, x+X);
      ObjectSet(name, OBJPROP_YDISTANCE, y+Y);
      ObjectSetInteger(0,name,OBJPROP_SELECTABLE,false);
      ObjectSetInteger(0,name,OBJPROP_SELECTED,false);
      ObjectSetInteger(0,name,OBJPROP_HIDDEN,true);
      ObjectSetInteger(0,name,OBJPROP_ANCHOR,align); 
   }
   ObjectSetText(name,Name,8,"Arial",clr);
}
//--------------------------------------------------------------------
void DrawHLINE(string name,double p, color clr=clrGray)
{
   if (ObjectFind(name)!=-1) ObjectDelete(name);
   ObjectCreate(name, OBJ_HLINE, 0, 0, p);
   ObjectSetInteger(0,name,OBJPROP_STYLE,0);
   ObjectSetInteger(0,name,OBJPROP_COLOR,clr);
   ObjectSetInteger(0,name,OBJPROP_SELECTABLE,false);
   ObjectSetInteger(0,name,OBJPROP_SELECTED,false);
   ObjectSetInteger(0,name,OBJPROP_HIDDEN,true);
}
//--------------------------------------------------------------------
void OnDeinit(const int reason)
{
   if (!IsTesting())
   {
      ObjectsDeleteAll(0,"cm");
   }
   Comment("");
   EventKillTimer();
}
//+------------------------------------------------------------------+
bool CloseByOrders()
{
   bool error=true;
   int b=0,s=0,TicketApponent=0,Ticket,OT,j,LaslApp=-1;
   int closeby=0;
   while(!IsStopped())
   {
      for (j = OrdersTotal()-1; j >= 0; j--)
      {
         if (OrderSelect(j, SELECT_BY_POS))
         {
            if (OrderSymbol() == Symbol() && (Magic==-1 || Magic==OrderMagicNumber()))
            {
               OT = OrderType();
               Ticket=OrderTicket();
               if (OT>1) {error=OrderDelete(Ticket);continue;}
               if (TicketApponent==0) {TicketApponent=Ticket;LaslApp=OT;}
               else
               {
                  if (LaslApp==OT) continue;
                  if (OrderCloseBy(Ticket,TicketApponent,Green)) TicketApponent=0;
                  else {closeby++;Print("Error",GetLastError()," close order N ",Ticket," <-> ",TicketApponent);}
               }
            }
         }
      }
      if (closeby>5) break;
      b=0;s=0;
      for (j = OrdersTotal()-1; j >= 0; j--)
      {                                               
         if (OrderSelect(j,SELECT_BY_POS,MODE_TRADES))
         {
            if (OrderSymbol() == Symbol() && (Magic==-1 || Magic==OrderMagicNumber()))
            {
               OT = OrderType();
               if (OT==OP_BUY) b++;
               if (OT==OP_SELL) s++;
            }
         }
      }
      if (b==0 || s==0) break;
   }
   CloseAll(-1);
   return(true);
}
//-------------------------------------------------------------------
bool CloseAll(int tip)
{
   bool error=true;
   int j,err,nn=0,OT;
   while(true)
   {
      for (j = OrdersTotal()-1; j >= 0; j--)
      {
         if (OrderSelect(j, SELECT_BY_POS))
         {
            if (OrderSymbol() == Symbol() && (Magic==-1 || Magic==OrderMagicNumber()) )
            {
               OT = OrderType();
               if (tip!=-1 && tip!=OT) continue;
               if (OT==OP_BUY) 
               {
                  error=OrderClose(OrderTicket(),OrderLots(),NormalizeDouble(Bid,Digits),slippage,Blue);
               }
               if (OT==OP_SELL) 
               {
                  error=OrderClose(OrderTicket(),OrderLots(),NormalizeDouble(Ask,Digits),slippage,Red);
               }
               if (!error) 
               {
                  err = GetLastError();
                  if (err<2) continue;
                  if (err==129) 
                  {
                     RefreshRates();
                     continue;
                  }
                  if (err==146) 
                  {
                     if (IsTradeContextBusy()) Sleep(2000);
                     continue;
                  }
                  Print("Error",err," close order N ",OrderTicket(),"     ",TimeToStr(TimeCurrent(),TIME_SECONDS));
               }
            }
         }
      }
      int n=0;
      for (j = 0; j < OrdersTotal(); j++)
      {
         if (OrderSelect(j, SELECT_BY_POS))
         {
            if (OrderSymbol() == Symbol() && (Magic==-1 || Magic==OrderMagicNumber()))
            {
               OT = OrderType();
               if (OT>1) 
               {
                  int Ticket=OrderTicket();
                  if (tip==-1) error=OrderDelete(Ticket);
                  else
                  {                  
                     if (tip==OP_BUY && (OT==OP_BUYLIMIT || OT==OP_BUYSTOP)) error=OrderDelete(Ticket);
                     if (tip==OP_SELL && (OT==OP_SELLLIMIT || OT==OP_SELLSTOP)) error=OrderDelete(Ticket);
                  }
                  continue;
               }
               if (tip!=-1 && tip!=OT) continue;
               n++;
            }
         }  
      }
      if (n==0) break;
      nn++;
      if (nn>10) 
      {
         Alert(Symbol()," Failed to close all deal, there is still ",n);
         return(false);
      }
      Sleep(1000);
      RefreshRates();
   }
   return(true);
}
//------------------------------ปุม-เมนู ปิดออร์ดอร์  CloseProfit bouttom------------------------------------
bool ButtonCreate(const long              chart_ID=0,               //Chart ID
                  const string            name="Button",            // button name
                  const int               sub_window=0,             // subwindow number
                  const long               x=0,                     // X-coordinate
                  const long               y=0,                     // Y-coordinate
                  const int               width=50,                 // button width
                  const int               height=18,                // button height
                  const string            text="Button",            // text
                  const string            font="Arial",             // font
                  const int               font_size=8,              // font size
                  const color             clr=clrBlack,             // text color
                  const color             clrON=clrLightGray,       // background color
                  const color             clrOFF=clrLightGray,      // background color
                  const color             border_clr=clrNONE,       // border color
                  const bool              state=false,       //
                  const ENUM_BASE_CORNER  CORNER=CORNER_RIGHT_UPPER)
  {
   if (ObjectFind(chart_ID,name)==-1)
   {
      ObjectCreate(chart_ID,name,OBJ_BUTTON,sub_window,0,0);
      ObjectSetInteger(chart_ID,name,OBJPROP_XSIZE,width);
      ObjectSetInteger(chart_ID,name,OBJPROP_YSIZE,height);
      ObjectSetInteger(chart_ID,name,OBJPROP_CORNER,CORNER);
      ObjectSetString(chart_ID,name,OBJPROP_FONT,font);
      ObjectSetInteger(chart_ID,name,OBJPROP_FONTSIZE,font_size);
      ObjectSetInteger(chart_ID,name,OBJPROP_BACK,0);
      ObjectSetInteger(chart_ID,name,OBJPROP_SELECTABLE,0);
      ObjectSetInteger(chart_ID,name,OBJPROP_SELECTED,0);
      ObjectSetInteger(chart_ID,name,OBJPROP_HIDDEN,1);
      ObjectSetInteger(chart_ID,name,OBJPROP_ZORDER,1);
      ObjectSetInteger(chart_ID,name,OBJPROP_STATE,state);
   }
   ObjectSetInteger(chart_ID,name,OBJPROP_BORDER_COLOR,border_clr);
   color back_clr;
   if (ObjectGetInteger(chart_ID,name,OBJPROP_STATE)) back_clr=clrON; else back_clr=clrOFF;
   ObjectSetInteger(chart_ID,name,OBJPROP_BGCOLOR,back_clr);
   ObjectSetInteger(chart_ID,name,OBJPROP_COLOR,clr);
   ObjectSetString(chart_ID,name,OBJPROP_TEXT,text);
   ObjectSetInteger(chart_ID,name,OBJPROP_XDISTANCE,x+X);
   ObjectSetInteger(chart_ID,name,OBJPROP_YDISTANCE,y+Y);
   return(true);
}
//--------------------------------------------------------------------
bool RectLabelCreate(const long             chart_ID=0,               // Chart ID
                     const string           name="RectLabel",         // label name
                     const int              sub_window=0,             // subwindow number
                     const long              x=0,                     // X-coordinate
                     const long              y=0,                     // Y-coordinate
                     const int              width=50,                 // width
                     const int              height=18,                // height
                     const color            back_clr=clrWhite,        // background color
                     const color            clr=clrBlack,             // flat border color (Flat)
                     const ENUM_LINE_STYLE  style=STYLE_SOLID,        // flat border style
                     const int              line_width=1,             // flat border thickness
                     const bool             back=false,               // on the background
                     const bool             selection=false,          // allocate for movement
                     const bool             hidden=true,              // hidden in the list of objects
                     const long             z_order=0)                // priority on mouse click
  {
   ResetLastError();
   if (ObjectFind(chart_ID,name)==-1)
   {
      ObjectCreate(chart_ID,name,OBJ_RECTANGLE_LABEL,sub_window,0,0);
      ObjectSetInteger(chart_ID,name,OBJPROP_BORDER_TYPE,BORDER_FLAT);
      ObjectSetInteger(chart_ID,name,OBJPROP_CORNER,CORNER_RIGHT_UPPER);
      ObjectSetInteger(chart_ID,name,OBJPROP_STYLE,style);
      ObjectSetInteger(chart_ID,name,OBJPROP_WIDTH,line_width);
      ObjectSetInteger(chart_ID,name,OBJPROP_BACK,back);
      ObjectSetInteger(chart_ID,name,OBJPROP_SELECTABLE,selection);
      ObjectSetInteger(chart_ID,name,OBJPROP_SELECTED,selection);
      ObjectSetInteger(chart_ID,name,OBJPROP_HIDDEN,hidden);
      ObjectSetInteger(chart_ID,name,OBJPROP_ZORDER,z_order);
      //ObjectSetInteger(chart_ID,name,OBJPROP_ALIGN,ALIGN_RIGHT); 
   }
   ObjectSetInteger(chart_ID,name,OBJPROP_BGCOLOR,back_clr);
   ObjectSetInteger(chart_ID,name,OBJPROP_COLOR,clr);
   ObjectSetInteger(chart_ID,name,OBJPROP_XSIZE,width);
   ObjectSetInteger(chart_ID,name,OBJPROP_YSIZE,height);
   ObjectSetInteger(chart_ID,name,OBJPROP_XDISTANCE,x+X);
   ObjectSetInteger(chart_ID,name,OBJPROP_YDISTANCE,y+Y);
   return(true);
}
//--------------------------------------------------------------------
string Error(int code)
{
   switch(code)
   {
      case 0:   return("No mistakes");
      case 1:   return("No mistake, but the result is unknown");                            
      case 2:   return("General error");                                                   
      case 3:   return("Invalid parameters");                                         
      case 4:   return("Trade server is busy");                                          
      case 5:   return("Old version of the client terminal");                            
      case 6:   return("No connection to the trade server");                                  
      case 7:   return("Insufficient rights");                                              
      case 8:   return("Too frequent requests");                                         
      case 9:   return("Invalid operation disrupting server operation");      
      case 64:  return("Account blocked");                                              
      case 65:  return("Invalid account number");                                       
      case 128: return("The waiting period has expired");                          
      case 129: return("Wrong price");                                              
      case 130: return("Wrong feet");                                             
      case 131: return("Incorrect volume");                                             
      case 132: return("Market closed");                                                   
      case 133: return("Trading is prohibited");                                               
      case 134: return("Not enough money to complete the transaction");                     
      case 135: return("Price has changed");                                                
      case 136: return("No prices");                                                        
      case 137: return("Broker is busy");                                                   
      case 138: return("New prices");                                                     
      case 139: return("The order is blocked and is already being processed");                        
      case 140: return("Only purchase allowed");                                       
      case 141: return("Too many requests");                                         
      case 145: return("Modification prohibited, as the order is too close to the market");    
      case 146: return("The trading subsystem is busy");                                     
      case 147: return("The use of the order expiration date is prohibited by the broker");         
      case 148: return("The number of open and pending orders has reached the limit, set by the broker.");
      default:   return(StringConcatenate("Error ",code," unknown "));
   }
}
//----------------------------------------------------------เมนู กำหนด จำนวน ออร์เดอร์ ระยะห่าง ด้านบน **---------
bool EditCreate(const long             chart_ID=0,               // Chart ID 
                const string           name="Edit",              // object name
                const int              sub_window=0,             // subwindow number 
                const int              x=0,                      // X-coordinate
                const int              y=0,                      // Y-coordinate
                const int              width=50,                 // width
                const int              height=18,                // height
                const string           text="Text",              // text
                const string           font="Arial",             // font
                const int              font_size=8,             // font size 
                const ENUM_ALIGN_MODE  align=ALIGN_RIGHT,       // alignment method 
                const bool             read_only=true,           // ability to edit
                const ENUM_BASE_CORNER corner=CORNER_RIGHT_UPPER, // corner of the graph to anchor
                const color            clr=clrRed,             // text color ........clrBlack
                const color            back_clr=clrWhite,        // background color
                const color            border_clr=clrBlack,       // border color .....clrBlack
                const bool             back=false,               // on the background
                const bool             selection=false,          // allocate for movement
                const bool             hidden=true,              // hidden in the list of objects
                const long             z_order=0)                // priority on mouse click
  { 
   ResetLastError(); 
   if(!ObjectCreate(chart_ID,name,OBJ_EDIT,sub_window,0,0)) 
     { 
      Print(__FUNCTION__, 
            ": failed to create object ",name,"! Error code = ",GetLastError()); 
      return(false); 
     } 
   ObjectSetInteger(chart_ID,name,OBJPROP_XDISTANCE,x+X); 
   ObjectSetInteger(chart_ID,name,OBJPROP_YDISTANCE,y+Y); 
   ObjectSetInteger(chart_ID,name,OBJPROP_XSIZE,width); 
   ObjectSetInteger(chart_ID,name,OBJPROP_YSIZE,height); 
   ObjectSetString(chart_ID,name,OBJPROP_TEXT,text); 
   ObjectSetString(chart_ID,name,OBJPROP_FONT,font); 
   ObjectSetInteger(chart_ID,name,OBJPROP_FONTSIZE,font_size); 
   ObjectSetInteger(chart_ID,name,OBJPROP_ALIGN,align); 
   ObjectSetInteger(chart_ID,name,OBJPROP_READONLY,read_only); 
   ObjectSetInteger(chart_ID,name,OBJPROP_CORNER,corner); 
   ObjectSetInteger(chart_ID,name,OBJPROP_COLOR,clr); 
   ObjectSetInteger(chart_ID,name,OBJPROP_BGCOLOR,back_clr); 
   ObjectSetInteger(chart_ID,name,OBJPROP_BORDER_COLOR,border_clr); 
   ObjectSetInteger(chart_ID,name,OBJPROP_BACK,back); 
   ObjectSetInteger(chart_ID,name,OBJPROP_SELECTABLE,selection); 
   ObjectSetInteger(chart_ID,name,OBJPROP_SELECTED,selection); 
   ObjectSetInteger(chart_ID,name,OBJPROP_HIDDEN,hidden); 
   ObjectSetInteger(chart_ID,name,OBJPROP_ZORDER,z_order); 
   return(true); 
  } 
//+------------------------------------------------------------------+ 
string Text(bool P,string a,string b)
{
   if (P) return(a);
   else return(b);
}
//------------------------------------------------------------------
void drawtext(string Name, datetime T1, double Y1, string lt,color c)
{
   ObjectDelete (Name);
   ObjectCreate (Name, OBJ_TEXT,0,T1,Y1,0,0,0,0);
   ObjectSetText(Name, lt,8,"Arial");
   ObjectSetInteger(0,Name, OBJPROP_COLOR, c);
   ObjectSetInteger(0,Name, OBJPROP_ANCHOR, ANCHOR_LOWER);
}

//--------------------------------------------------------------------
