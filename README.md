# hello-world
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using System.Web.UI;
using System.Web.UI.WebControls;
using System.Data;
using System.Text.RegularExpressions;
using System.Text;

using iTextSharp.text.pdf;
using iTextSharp.text.pdf.parser;
using System.IO;
using System.Collections;
using RAS;


namespace RASystem
{
    public partial class choicemaster : System.Web.UI.Page
    {
        int? iddept;
        const string Applied = "Applied";
        protected void Page_Load(object sender, EventArgs e)
        {
            SetCalender();
            Response.Cache.SetCacheability(System.Web.HttpCacheability.NoCache);
            Response.Cache.SetNoStore();

            if (IsPostBack == false)
            {
                //FillFundSource(DDLFundSrc,0,"");

               
                    myTAB.TabIndex = 0;


                    RAS.RASDomainD myr = new RAS.RASDomainD();
                    System.Linq.IQueryable<RAS.personalinfo> pinf;
                    pinf = myr.GetPersonalinfoes(Convert.ToInt32(Session["UID"].ToString()));
                    RAS.personalinfo itm = new RAS.personalinfo();
                    ToggleTab(false);
                    TxtEmail1.Text = Session["Email"].ToString();
                    mySQLConnection.ErrorReport("Getting personal info");
                    try
                    {
                        foreach (RAS.personalinfo itmx in pinf)
                        {
                                Session.Add("idPinf", itmx.idPersonalInfo);
                                TxtEmpNo.Text = itmx.EmpNo;
                                TxtEmpName.Text = itmx.EmpName;
                                if (itmx.p_Email.Equals("") == true)
                                    TxtEmail1.Text = Session["Email"].ToString();
                                else
                                    TxtEmail1.Text = itmx.p_Email;
                                TxtMobile.Text = itmx.Mobile;
                                TxtTelephone.Text = itmx.Extention;
                                TxtPassport.Text = itmx.PassportNo;
                                TxtAccount.Text = itmx.AccountNo;
                                if (Convert.ToBoolean(itmx.VisaReq) == true)
                                    RdoVisaY.Checked = true;
                                else
                                    RdoVisaN.Checked = true;
                                TxtAddress.Text = itmx.Address;
                                //Session["WKLOC"]= itmx.idWorkLocation;

                                if (itmx.Department_idDepartment != null)
                                    iddept = (Int32)itmx.Department_idDepartment.Value;
                                ChkConfirm.Checked = (bool)itmx.confirmBankinfo;
                                itm = itmx;
                                ToggleTab(true);
                            }

                         }
                        catch (Exception ex)
                        {
                    
                          mySQLConnection.ErrorReport(ex.ToString());
                        }

                    BindCombo(DDLWorkArea, "select * from worklocation", "idWorkLocation", "WorkLocationName");
                    PositionCombo(itm.idWorkLocation, DDLWorkArea);


                    PositionCombo(itm.College_idCollege, DDLCollegeApp);


                    BindCombo(DDLNation, "select idcountry,nationality from country order by nationality", "idcountry", "nationality");
                    PositionCombo(itm.Nationality, DDLNation);
                    RequiredFieldValidator8.InitialValue = GetBlankValue(DDLNation);

                    BindCombo(DDLJobTitle, "select idjob_title,jobtitle from job_title order by jobtitle", "idjob_title", "jobtitle");
                    PositionCombo(itm.Job_Title_idJob_Title, DDLJobTitle);
                    RequiredFieldValidator9.InitialValue = GetBlankValue(DDLJobTitle);


                    BindCombo(DDLCollege, "select idCollege,collegeName from college order by collegeName", "idCollege", "collegeName");
                    PositionCombo(itm.College_idCollege, DDLCollege);
                    RequiredFieldValidator1.InitialValue = GetBlankValue(DDLCollege);
                    // position data got from College
                    PositionRefData(itm, DDLDept, "select * from department where iddepartment=" + itm.Department_idDepartment, "College_idCollege", "DepartmentName", itm.Department_idDepartment);

                    BindCombo(DDLDept, "select iddepartment,departmentname from department where college_idCollege=" + DDLCollege.SelectedValue + " order by departmentname", "iddepartment", "departmentname");
                    PositionCombo(itm.Department_idDepartment, DDLDept);

                    BindCombo(DDLBank, "Select * from BankMaster order by BankName", "idBankMaster", "BankName");
                    PositionCombo(itm.BankName_IDBank, DDLBank);

                    // position data got from bank
                    PositionRefData(itm, DDLBranch, "select * from bankbranch where bankmasteridbankmast=" + DDLBank.SelectedValue + " order by branchname", "idBankBranch", "BranchName", Convert.ToInt32(itm.BranchName_IdBranch));

                    BindCombo(DDLQualif, "select * from Qualification order by qualification", "idqualification", "Qualification");
                    PositionCombo(itm.Qualification, DDLQualif);

                    BindCombo(DDLEmpStatus, "select * from EmpStatus order by Emp_status", "idnew_table", "Emp_Status");
                    PositionCombo(itm.ID_Emp_Status, DDLEmpStatus);

                    BindCombo(DDLEmpType, "select * from emptype order by emptype", "idEmpType", "emptype");

                    PositionCombo(itm.Department_idDepartment, DDLDepartment);

                    if (itm.confirmBankinfo != null)
                        ChkConfirm.Checked = (bool)itm.confirmBankinfo;
                    //PositionCombo(itm.EmpType_IDEmpType, DDLEmpType);
                    if (DDLEmpType.SelectedItem.Text.Equals("Full Time", StringComparison.InvariantCultureIgnoreCase) == true)
                    {
                        RDOOnReq1.Enabled = false;
                        RDOOnReq2.Enabled = false;
                        RDOOnReq3.Enabled = false;
                        TxtOnReqTxt.Enabled = false;
                    }


                    if (itm.idno != null)
                    {
                        BindGridData();
                    }



                    RadioButton1.Checked = true;
                    ShowImage();

                    BindjobDetail("-1");
                    DisableStaff();
            
            }
            paymentoptionEnDis();
            
        }

        private void BindGridData()
        {
            DataSet mydt = new DataSet();
            mydt = mySQLConnection.getmySQLdataset("select IdApplication as ID, date_format( S_date,'%d-%M-%y') as DateStart , date_format(E_Date,'%d-%M-%y') as DateEnd, CurrentStatus from application where personalinfo_idpersonalinfo=" + Session["idPinf"].ToString() + " order by applicationdate desc ");
            GridView1.DataSource = mydt;
            GridView1.DataBind();
        }

        private void ToggleTab(bool vlaue)
        {
            try
            {
                tabpanelupld.Visible = vlaue;
                _application.Visible = vlaue;
                if (Session["CT"] == "2")
                    myTAB.ActiveTabIndex = 2;
            }
            catch (Exception)
            {}
        }

        private void FillFundSource(DropDownList ddl,int index, string olddta)
        {
            try
            {
                ddl.Items.Clear();
                RAS.GrantWebData.WebGrantSoapClient mygrant = new RAS.GrantWebData.WebGrantSoapClient();
                List<string> dta = mygrant.GetProject(index, olddta);
                foreach (string itmx in dta)
                {
                    ddl.Items.Add(itmx);
                }
            }
            catch (Exception)
            {}
        }

        private void SetCalender()
        {
            CalendarExtender2.StartDate = DateTime.Now;
            CalendarExtender2.EndDate = DateTime.Now.AddYears(1);;

            CalendarExtender1.EndDate = DateTime.Now.AddYears(3);
            CalendarExtender1.StartDate = DateTime.Now;

            
        }

        //private void GetSourceFundingData()
        //{
        //    RAS.GrantWebData.WebGrantSoapClient myGrant = new RAS.GrantWebData.WebGrantSoapClient();
        //    List<string> mylst = myGrant.WebGrantSrv();
        //    foreach (string grnt in mylst)
        //    {
        //        string[] dta = Regex.Split(grnt, "#");
        //        ListItem mlst = new ListItem();
        //        mlst.Value = dta[0];
        //        mlst.Text = dta[1];
        //        DDLFundSrc.Items.Add(mlst);
        //    }
        //    DDLFundSrc.SelectedIndex = -1;
        //}

        private void PositionRefData(RAS.personalinfo itm, DropDownList ddl, string sql, string colName, string dispcol, int? matchvalue)
        {

            try
            {
                //mySQLConnection.ErrorReport("Position Combo " + ddl.ID.ToString());
                DataSet dt = new DataSet();
                dt = mySQLConnection.getmySQLdataset(sql);

                BindCombo(ddl, sql, colName, dispcol);
                if (dt.Tables[0].Rows.Count > 0)
                {
                    int a = 0;
                    foreach (ListItem it in ddl.Items)
                    {
                        if (Convert.ToInt32(it.Value) == matchvalue)
                        {
                            ddl.SelectedIndex = a;
                            break;
                        }
                        a++;
                    }
                }
            }
            catch (Exception )
            {}
        }

        private void BindCombo(DropDownList ddl,string sql,string value,string text)
        {
            //mySQLConnection.ErrorReport("Bind Combo " + ddl.ID.ToString());
            ddl.Items.Clear();
            DataSet dtnation = new DataSet();
            dtnation = mySQLConnection.getmySQLdataset(sql);
            ddl.DataSource = dtnation;
            ddl.DataValueField = value;
            ddl.DataTextField = text;
            ddl.DataBind();
        }
        private void PositionCollege()
        {
            
            for (int a = 0; a < DDLCollege.Items.Count; a++)
            {
                DDLCollege.SelectedIndex = a;
                int x = 0;
                foreach (ListItem it in DDLDept.Items)
                {
                    if (Convert.ToInt32(it.Value) == iddept)
                    {
                        DDLDept.SelectedIndex = x;
                        return;
                    }
                    x++;
                }
            }
        }
        private void PositionCombo(int? searchvalue, DropDownList ddl)
        {
            int a = 0;
            
            foreach (ListItem it in ddl.Items )
            {
                if (Convert.ToInt32( it.Value)== searchvalue )
                {
                    ddl.SelectedIndex = a;
                    return;
                }
                a++;
            }
            
        }

        private string GetBlankValue( DropDownList ddl)
        {
            int a = 0;

            foreach (ListItem it in ddl.Items)
            {
                if (it.Text.Trim().Equals("")==true)
                {

                    return  it.Value;
                }
                a++;
            }
            return "";
        }

        protected void BtnSave_Click(object sender, EventArgs e)
        {
            if (ChkConfirm.Checked == false)
            {
                ShowPopUpMsg("Please confirm bank information.");
                return;
            }
            bool vsa = RdoVisaY.Checked;       

            DataSet mydt = new DataSet();
            mydt = mySQLConnection.getmySQLdataset("select * from personalinfo where idno=" + Session["UID"].ToString());
            RAS.RASDomainD myr = new RAS.RASDomainD();
            
            string OnReq;

            if (RDOOnReq1.Checked == true)
                OnReq = "2";
            else if (RDOOnReq2.Checked == true)
                OnReq = "3";
            else
                OnReq = TxtOnReqTxt.Text;

            if (mydt.Tables[0].Rows.Count == 1)
            {
                string sql = "   UPDATE personalinfo  SET idWorkLocation=" + DDLWorkArea.SelectedValue +", EmpNo ='" + TxtEmpNo.Text + "',EmpName ='" + TxtEmpName.Text + "',Mobile ='" + TxtMobile.Text + "',Extention ='" + TxtTelephone.Text + "',p_Email ='" + TxtEmail1.Text + "', Job_Title_idJob_Title = " + DDLJobTitle.SelectedValue + ",College_idCollege=" + DDLCollege.SelectedValue + ", Department_idDepartment = " + DDLDept.SelectedValue + ",Nationality = " + DDLNation.SelectedValue + ",BankName_IDBank=" + DDLBank.SelectedValue + ", BranchName_idBranch=" + DDLBranch.SelectedValue + ",Qualification=" + DDLQualif.SelectedValue + ", VisaReq = " + vsa + ",Address='" + TxtAddress.Text + "', accountno='" + TxtAccount.Text + "', passportno = '" + TxtPassport.Text + "', ID_Emp_Status=" + DDLEmpStatus.SelectedValue + ",confirmBankinfo=" + ChkConfirm.Checked +"    WHERE idno = " + Session["UID"].ToString();
                sql = Regex.Replace(sql, "''", "null");
                sql = Regex.Replace(sql, "=,", "=null,");
                mySQLConnection.ExecutemySQLData(sql);
                //mySQLConnection.ExecutemySQLData("update personalinfo set College_idCollege=" + DDLCollege.SelectedValue + ",Department_idDepartment=" + DDLDepartment.SelectedValue + " WHERE idno = " + Session["UID"].ToString());
            }
            else
            {
                InsertDta(vsa, "", "null");
            }
            ShowPopUpMsg("Information has been saved.");
            Session.Add("CT","2");
            Response.Redirect("~/choicemaster.aspx");
        }

        private void InsertDta(bool vsa, string contStart, string contEnd)
        {
            string _branch;
            if (DDLBranch.SelectedItem != null)
                _branch = DDLBranch.SelectedItem.Value;
            else
                _branch = "null";
            mySQLConnection.ExecutemySQLData("insert into personalinfo values(null," + Session["UID"].ToString() + ",'" + TxtEmpNo.Text + "','" + TxtEmpName.Text + "','" + TxtMobile.Text + "','" + TxtTelephone.Text + "','" + TxtEmail1.Text + "'," + Convert.ToInt32(DDLJobTitle.SelectedValue) + ","+ Convert.ToInt32(DDLNation.SelectedValue) + ",'" + TxtPassport.Text + "'," + vsa + ",'" + TxtAddress.Text + "','" + TxtAccount.Text + "'," + DDLBank.SelectedItem.Value + "," + _branch + "," + DDLQualif.SelectedValue + "," + DDLEmpStatus.SelectedValue +"," + Convert.ToInt32( DDLCollege.SelectedValue ) + "," + Convert.ToInt32(DDLDept.SelectedValue) + ","+ ChkConfirm.Checked +"," + DDLWorkArea.SelectedValue +  ")");
            DataSet mydt = new DataSet();
            mydt = mySQLConnection.getmySQLdataset("select * from personalinfo where empno = " + Session["UID"].ToString());
            if (mydt.Tables[0].Rows.Count==1)
                Session.Add("idPinf", mydt.Tables[0].Rows[0]["idno"].ToString());
        }

        protected void DDLCollege_SelectedIndexChanged(object sender, EventArgs e)
        {
            BindCombo(DDLDept, "select iddepartment,departmentname from department where college_idCollege=" + DDLCollege.SelectedValue + " order by departmentname", "iddepartment", "departmentname");
            PositionCombo(iddept, DDLDept);
        }

        protected void DDLBank_SelectedIndexChanged(object sender, EventArgs e)
        {

            BindCombo(DDLBranch, "select * from bankbranch where bankmasteridbankmast=" + DDLBank.SelectedValue + " order by branchname", "idbankbranch", "branchname");
            
        }

        protected void Btn_NewApp_Click(object sender, EventArgs e)
        {
            //if (GetUnderProcessApplication() == true) return;
            GridView1.SelectedIndex = -1;
            PnlNewApp.Enabled = true;
            //TxtTermsCond.Text = mySQLConnection.ReadFile("TermsCond.html"); ;
            TxtSpec.Text = "";
            //TxtJobDet.Text = "";
            TxtFulTMPTM.Text = "";
            RDOOnReq1.Checked = false;
            RDOOnReq2.Checked = false;
            RDOOnReq3.Checked = false;
            TxtOnReqTxt.Text = "";
            TxtContractEnd.Text = "";
            TxtContractStart.Text = "";
            TxtContraP.Text = "";
            TxtJobTask1.Text = "";
            TxtJobTask2.Text = "";
            TxtJobTask3.Text = "";
            TxtJobTask4.Text = "";

            //SetVFundsource(true);
            LblSourceFund.Text = "";
            
            Btn_SaveApp.Enabled = true;
            BindjobDetail("-1");
            //ChkSendApproval.Checked = false;
            Btn_SaveApp.Enabled = true;
            Btn_SaveFinalSend.Enabled = true;
            Panel1.Enabled = false;
        }
        private void ShowPopUpMsg(string msg)
        {
            StringBuilder sb = new StringBuilder();
            sb.Append("alert('");
            sb.Append(msg.Replace("\n", "\\n").Replace("\r", "").Replace("'", "\\'"));
            sb.Append("');");
            ScriptManager.RegisterStartupScript(this.Page, this.GetType(), "showalert", sb.ToString(), true);
        }
        protected void Btn_SaveApp_Click(object sender, EventArgs e)
        {


           
        }

        private void SaveApp(bool Send)
        {
            if (TxtContractStart.Text == TxtContractEnd.Text)
            {
                ShowPopUpMsg("Please check contact Dates.");
                return;
            }
            //if (GetUnderProcessApplication() == true) return;
            DataSet mydt = new DataSet();
            RAS.GrantWebData.WebGrantSoapClient mygr = new RAS.GrantWebData.WebGrantSoapClient();
            string dt = DDLSourceFund.SelectedItem.Text;// GetProjectComboInfo();
            string pnumber = mygr.GetProjnumber("'" + dt + "'");
            string[] PIInfo = Regex.Split(mygr.GetPIDetail("'" + dt + "'"), "####");
            if (PIInfo.Length == 0)
            {
                ShowPopUpMsg("Invlid PI Information retrived. Please check the Grant Project hosted on 216");
                return;
            }
            string onreq = "";
            if (RDOOnReq1.Checked == true)
                onreq = "2";
            else if (RDOOnReq2.Checked == true)
                onreq = "3";
            else
                onreq = TxtOnReqTxt.Text;

            string sql = "";
            bool isresending = false;
            GridViewRow grv = GridView1.SelectedRow;
            if (grv != null &&  Send == true)
            {
                if (grv.Cells[5].Text == "Returned") isresending = true;
                pnumber = mygr.GetProjnumber("'" + LblSourceFund.Text + "'");
                string id = grv.Cells[2].Text;
                string applied = "Not Applied";
                if (Send == true) applied = "Applied";
                sql = "update application set s_date='" + Convert.ToDateTime(TxtContractStart.Text).ToString("yyyy-MM-dd") + "',sendforApproval=" + Send + ", e_date='" + Convert.ToDateTime(TxtContractEnd.Text).ToString("yyyy-MM-dd") + "',specificCondition='" + TxtSpec.Text + "', sourceoffund='" + pnumber + "', contractperiod='" + TxtContraP.Text + "', onreq='" + onreq + "', ft_ptpayment='" + TxtFulTMPTM.Text + "', currentstatus='" + applied + "', college_idCollege=" + DDLCollege.SelectedValue  + ", Department_idDepartment=" + DDLDepartment.SelectedValue + "  where idapplication=" + id;
                mySQLConnection.ExecutemySQLData(sql);
                PIInfo = Regex.Split(mygr.GetPIDetail("'" + LblSourceFund.Text + "'"), "####");
                goto ResendApp;
            }


            sql = "insert into application values(null,null,'" + Convert.ToDateTime(TxtContractStart.Text).ToString("yyyy-MM-dd") + "','" + Convert.ToDateTime(TxtContractEnd.Text).ToString("yyyy-MM-dd") + "','" + DateTime.Now.ToString("yyyy-MM-dd") + "','','" + TxtSpec.Text + "','Not Applied',null," + Session["idPinf"].ToString() + "," + Send + "," + pnumber + ",0,'" + TxtContraP.Text + "','','" + onreq + "','" + TxtFulTMPTM.Text + "','" + PIInfo[1] + "'," + DDLEmpType.SelectedValue + "," + DDLCollegeApp.SelectedValue + "," + DDLDepartment.SelectedValue + " )";
            mySQLConnection.ExecutemySQLData(sql);
        ResendApp:
            //================ getting id application
            mydt = mySQLConnection.getmySQLdataset("select max(idapplication) from application where personalinfo_idpersonalinfo=" + Session["idPinf"].ToString());

            int appid = Convert.ToInt32("0" + mydt.Tables[0].Rows[0][0].ToString());


            if (Send == true)
            {
                //================ Sending mail as confirmation in case of new New or Returned 
                string appli = mySQLConnection.ReadFile("Applicant.html");
                appli = Regex.Replace(appli, "<Name>", TxtEmpName.Text, RegexOptions.IgnoreCase);
                mySQLConnection.SendMail(TxtEmail1.Text, "Confirmation Message from Research Administration System", appli);
                appli = mySQLConnection.ReadFile("HODs.html");
                string mailid = PIInfo[1] + "@squ.edu.om";
                mailid = Regex.Replace(mailid, "@squ.edu.om@squ.edu.om", "@squ.edu.om");
                mySQLConnection.SendMail(mailid, "Research Administration System - Application of " + TxtEmpName.Text, appli);
            }


            sql = "update  personalinfo set College_idCollege=" + DDLCollegeApp.SelectedValue + ", department_iddepartment=" + DDLDepartment.SelectedValue + " where  idPersonalInfo=" + Session["idPinf"].ToString();
            mySQLConnection.ExecutemySQLData(sql);


            mydt = mySQLConnection.getmySQLdataset("select * from job_details where idapplication=" + appid);
            if (mydt.Tables[0].Rows.Count == 0)
            {
                for (int mlp = 1; mlp < 5; mlp++)
                {
                    TextBox mytb = (TextBox)PnlNewApp.FindControl("TxtJobTask" + mlp);
                    sql = "insert into job_details (jobtask,idapplication) values('" + mytb.Text + "'," + appid + ");";
                    //+"insert into job_details (jobtask,idapplication) values('" + TxtJobTask2.Text + "'," + appid + ");";
                    //sql = sql + "insert into job_details (jobtask,idapplication) values('" + TxtJobTask3.Text + "'," + appid + ");" + "insert into job_details (jobtask,idapplication) values('" + TxtJobTask4.Text + "'," + appid + ");";
                    mySQLConnection.ExecutemySQLData(sql);
                }
            }
            else
            {
                for (int mlp = 1; mlp < 5; mlp++)
                {
                    TextBox mytxt = (TextBox)PnlNewApp.FindControl("TxtJobTask" + mlp);
                    string recid="";
                    if (mlp == 1) recid = H1.Value;
                    if (mlp == 2) recid = H2.Value;
                    if (mlp == 3) recid = H3.Value;
                    if (mlp == 4) recid = H4.Value;
                    sql = "update job_details set jobtask='" + mytxt.Text + "' where idjob_details=" + recid + " and  idapplication = " + appid ;
                    if (recid == "")
                        sql = "insert into job_details (jobtask,idapplication) values('" + mytxt.Text + "'," + appid + ");";
                    
                    mySQLConnection.ExecutemySQLData(sql);
                }
            }

            ShowPopUpMsg("Information has been saved.");
            Response.Redirect("~/choicemaster.aspx");
        }

        //private string GetProjectComboInfo()
        //{
        //    try
        //    {
        //        string dt = DDLFundSrc.SelectedItem.Text + "/" + DDLFundSrc0.SelectedItem.Text + "/";
        //        if (DDLFundSrc1.SelectedItem.Text != "") dt = dt + DDLFundSrc1.SelectedItem.Text + "/";
        //        if (DDLFundSrc2.SelectedItem.Text != "") dt = dt + DDLFundSrc2.SelectedItem.Text + "/";
        //        if (DDLFundSrc3.SelectedItem.Text != "") dt = dt + DDLFundSrc3.SelectedItem.Text;
        //        return dt;
        //    }
        //    catch (Exception)
        //    { return ""; }
        //}

        private bool GetUnderProcessApplication()
        {
            DataSet mydt = new DataSet();
            mydt = mySQLConnection.getmySQLdataset("select count(idapplication) from application where currentstatus <> 'Approved' and  personalinfo_idpersonalinfo=" + Session["idPinf"].ToString());
            if (mydt.Tables[0].Rows[0][0].ToString() != "0")
            {
                ShowPopUpMsg("Application is already in pending mode. You can't apply further.");
                return true;
            }
            return false;
        }
        protected int ImageDirtoPDF(string PDFFile, string Folder)
        {
            if (File.Exists(PDFFile) == true)
                File.Delete(PDFFile);
            
            iTextSharp.text.Document Doc = new iTextSharp.text.Document(iTextSharp.text.PageSize.LETTER);
            string PDFOutput = Path.Combine(Path.Combine(Server.MapPath(" ") + "\\" + "imagesDir\\" + Session["UID"].ToString()), PDFFile);
            PdfWriter writer = PdfWriter.GetInstance(Doc, new FileStream(PDFOutput, FileMode.Create, FileAccess.Write, FileShare.Read));
            //Open the PDF for writing
            Doc.Open();
            //string Folder = "C:\\Images";
            int pgno = 0;
            foreach (string F in System.IO.Directory.GetFiles(Folder, "*.jpg"))
            {
                //Insert a page
                Doc.NewPage();
                iTextSharp.text.Image jpg = iTextSharp.text.Image.GetInstance(F);
                jpg.ScaleToFit(Doc.PageSize.Width, Doc.PageSize.Height);
                jpg.SpacingBefore = 10f;
                jpg.SpacingAfter = 1f;
                jpg.Alignment =iTextSharp.text.Element.ALIGN_CENTER;
                Doc.Add(jpg);
                pgno++;
            }
            if (pgno >0)
            Doc.Close();
            return pgno;
        }
        protected void AttachPDF(string PDFFile1, string PDFFile2,string Folder)
        {
            iTextSharp.text.Document Doc = new iTextSharp.text.Document(iTextSharp.text.PageSize.LETTER);
            string PDFOutput = Path.Combine(Path.Combine(Server.MapPath(" ") + "\\" + "imagesDir\\" + Session["UID"].ToString()), "tempX.pdf");
            PdfWriter writer = PdfWriter.GetInstance(Doc, new FileStream(PDFOutput, FileMode.Create, FileAccess.Write, FileShare.Read));
            //Open the PDF for writing
            Doc.Open();
            //string Folder = "C:\\Images";
            foreach (string F in System.IO.Directory.GetFiles(Folder, "*.pdf"))
            {
                //Insert a page
                Doc.NewPage();
                Doc.Add(new iTextSharp.text.Jpeg(new Uri(new FileInfo(F).FullName)));
            }
            Doc.Close();
        }
        public string WriteFile(string Filename)
        {
            
            RAS.GrantWebData.WebGrantSoapClient mypi = new RAS.GrantWebData.WebGrantSoapClient();
            GridViewRow grv = GridView1.SelectedRow;
            if (grv == null)
            {
                ShowPopUpMsg("Please select a record to print/");
                return "";
            }
            string[] pi = Regex.Split(mypi.GetPIDetail("'" + LblSourceFund.Text + "'"), "####"); ; ;// mypi.GetProjectString(grv.Cells[2].Text);//  mypi.GetPI(Convert.ToInt32( DDLFundSrc.SelectedItem.Value));
            //pi = mypi.GetPIDetail(LblSourceFund.Text);
            string folder = Path.GetFullPath(Path.Combine(System.AppDomain.CurrentDomain.BaseDirectory, "CForm"));
            Filename = folder + "//" + Filename;
            string newFile = folder + "//1.pdf"  ;

            Response.ContentType = "application/pdf";
            Response.AppendHeader("Content-Disposition","attachment;filename=itext_fill.pdf");
            PdfReader pdfreader = new PdfReader(Filename, null);
            using (PdfStamper ps = new PdfStamper(pdfreader, Response.OutputStream))
            {
                AcroFields at = ps.AcroFields;
                at.SetField("DateApp", DateTime.Now.ToString("dd-MMM-yyyy"));
                at.SetField("PIName", pi[0]);
                at.SetField("Nation", DDLNation.SelectedItem.Text);

                at.SetField("Tele", TxtTelephone.Text);
                at.SetField("EmpName", TxtEmpName.Text);
                at.SetField("EmpName2", TxtEmpName.Text);

                at.SetField("specCond", TxtSpec.Text);
                at.SetField("EmpNo", TxtEmpNo.Text);
                at.SetField("Email", TxtEmail1.Text);
                at.SetField("AreaWork", DDLWorkArea.SelectedItem.Text);
                at.SetField("EmpStat", DDLEmpStatus.SelectedItem.Text);
                at.SetField("Pass", TxtPassport.Text);
                at.SetField("EmpNation", DDLNation.SelectedItem.Text);
                at.SetField("Col", DDLCollege.SelectedItem.Text);
                at.SetField("Dept", DDLDept.SelectedItem.Text);
                if (RdoVisaY.Checked== true )
                    at.SetField("Visa", "Yes");
                else
                    at.SetField("Visa", "No");
                at.SetField("Quali", DDLQualif.SelectedItem.Text);
                at.SetField("JobTitl", DDLJobTitle.SelectedItem.Text);
                at.SetField("Address", TxtAddress.Text);
                at.SetField("Tele", TxtTelephone.Text);
                at.SetField("Mob", TxtMobile.Text);
                at.SetField("Bank", DDLBank.SelectedItem.Text);
                if (DDLBranch.SelectedItem != null)
                    at.SetField("Branch", DDLBranch.SelectedItem.Text);
                at.SetField("Account", TxtAccount.Text);
                at.SetField("EmpEmployment", DDLEmpType.SelectedItem.Text);

                at.SetField("EmpContPer", TxtContraP.Text);
                at.SetField("sdate", TxtContractStart.Text);
                at.SetField("edate", TxtContractEnd.Text);
               // at.SetField("EmpSourceFund", DDLFundSrc.SelectedItem.Text);
                
                at.SetField("ftime", TxtFulTMPTM.Text + "   RO per month.");

                at.SetField("title", LBLFundDetail.Text);
                at.SetField("pnumber", LblSourceFund.Text);

                if (RDOOnReq1.Checked == true)
                    at.SetField("ptime", "2/ R.O.");
                else if (RDOOnReq2.Checked == true)
                    at.SetField("ptime", "3/ R.O.");
                else
                {

                    at.SetField("ptime", TxtOnReqTxt.Text + "R.O.");
                }
                //at.SetField("EmpGenCon",TxtTermsCond.Text); 
                at.SetField("EmpSpeCon", TxtSpec.Text);
                //at.SetField("EmpOnRequest", "3/ R.O.");
                
                for (int lp = 1; lp <5; lp++)
                {
                    TextBox mytxt = (TextBox)PnlNewApp.FindControl("TxtJobTask" + lp);
                    at.SetField("jn" + lp.ToString(), lp.ToString());
                    at.SetField("jt" + lp.ToString(), mytxt.Text);
                }
                

                
                string id = grv.Cells[2].Text;
                DataSet mydtx = mySQLConnection.getmySQLdataset("select * from ApprovalFlowDetail where id_application=" + id);
                for (int ro = 0; ro < mydtx.Tables[0].Rows.Count; ro++)
                {
                    int pos = ro + 1;
                    at.SetField("l" + pos.ToString(), mydtx.Tables[0].Rows[ro]["Position"].ToString());
                    string act =  mydtx.Tables[0].Rows[ro]["Action"].ToString();
                    act = Regex.Replace(act,"A","Approved");
                    act = Regex.Replace(act,"C","Cancelled");
                    at.SetField("a" + pos.ToString(), act);
                    string  dt = Convert.ToDateTime( mydtx.Tables[0].Rows[ro]["ActionDate"].ToString()).ToString("dd-MMMM-yyyy");
                    at.SetField("d" + pos.ToString(), dt);
                }

                ps.FormFlattening = true;
            }

            Response.End();
            
            pdfreader.Close();
            return "";
        }
        public string ReadFile(string Filename)
        {


            string folder = Path.GetFullPath(Path.Combine(System.AppDomain.CurrentDomain.BaseDirectory, "CForm"));
            Filename = folder + "//" + Filename;
            PdfReader pdfreader = new PdfReader(Filename);
            string pdfText = string.Empty;

            for (int i = 1; i <= pdfreader.NumberOfPages; i++)
            {
                ITextExtractionStrategy itextextStrat = new SimpleTextExtractionStrategy();//; = new pdf.parser.SimpleTextExtractionStrategy();
                PdfReader reader = new PdfReader(Filename);
                String extractText = PdfTextExtractor.GetTextFromPage(reader, i, itextextStrat);
                
                extractText = Encoding.UTF8.GetString(ASCIIEncoding.Convert(Encoding.Default, Encoding.UTF8, Encoding.Default.GetBytes(extractText)));
                pdfText = pdfText + extractText;
                reader.Close();
            }
            return pdfText;
        }
        protected void GridView1_SelectedIndexChanged(object sender, EventArgs e)
        {
            //Btn_SaveApp.Enabled = true;
            //Btn_SaveFinalSend.Enabled = true;
            LblFileLocation.Text = "";
            GridViewRow grv = GridView1.SelectedRow;
            string id = grv.Cells[2].Text;

            BindjobDetail(id);


            DataSet dt = new DataSet();
            dt = mySQLConnection.getmySQLdataset("select myemailid,Position from filelevelinfodetail where apporovallevel=orderlevel and  idapplication=" + id);
            if (dt.Tables[0].Rows.Count > 0)
                LblFileLocation.Text = "File at :- " + dt.Tables[0].Rows[0]["myemailid"].ToString() + "@squ.edu.om     " + dt.Tables[0].Rows[0]["Position"].ToString();
            else
                LblFileLocation.Text = "No action has been taken by your PI yet.";

            if (grv.Cells[5].Text.Equals("Approv") == true)
            {
                Btn_SaveApp.Enabled = false;
                Btn_SaveFinalSend.Enabled = false;
                PnlNewApp.Enabled = false;
            }
            else if (grv.Cells[5].Text.Equals("Approved") == true)
            {
                Btn_SaveApp.Enabled = false;
                Btn_SaveFinalSend.Enabled = false;
                PnlNewApp.Enabled = false;
                LblFileLocation.Text = "";
            }
            else if (grv.Cells[5].Text.Equals("Cancelled") == true)
            {
                ImgBtn_PrintForm.Enabled = true;
                ImgBtn_PrintAttach.Enabled = true;
                Btn_SaveApp.Enabled = false;
                Btn_SaveFinalSend.Enabled = false;
                PnlNewApp.Enabled = false;
            }
            else if (grv.Cells[5].Text.Equals("Not Applied") == true)
            {
                Btn_SaveApp.Enabled = true;
                Btn_SaveFinalSend.Enabled = true;
            }
            else
            {
                //Btn_PrintForm.Enabled = false;
                //Btn_PrintAttach.Enabled = false;
            }
            
            if (grv.Cells[5].Text.Equals("Applied") == true || grv.Cells[5].Text.Equals("Not Applied"))
            {
                PnlNewApp.Enabled = true;
                //Btn_SaveApp.Enabled = true;
            }
            if (grv.Cells[5].Text.Equals("Returned") == true)
            {
                Btn_SaveApp.Enabled = true;
                Btn_SaveFinalSend.Enabled = true;
                PnlNewApp.Enabled = true;
            }
            
            
           
            dt = mySQLConnection.getmySQLdataset("Select * from application where idapplication=" +id );
            if (dt.Tables[0].Rows.Count == 1)
            {
                int idcol = Convert.ToInt32(dt.Tables[0].Rows[0]["College_idCollege"].ToString());
                
                PositionCombo(idcol, DDLCollegeApp);
                TxtContractStart.Text =Convert.ToDateTime( dt.Tables[0].Rows[0]["S_date"].ToString()).ToString("dd-MMM-yyyy");
                TxtContractEnd.Text =Convert.ToDateTime( dt.Tables[0].Rows[0]["E_date"].ToString()).ToString("dd-MMM-yyyy");
                //TxtTermsCond.Text = dt.Tables[0].Rows[0]["TermsCondition"].ToString();
                TxtSpec.Text =  dt.Tables[0].Rows[0]["SpecificCondition"].ToString();
                //TxtJobDet.Text = dt.Tables[0].Rows[0]["SpecificCondition"].ToString();
                TxtContraP.Text = dt.Tables[0].Rows[0]["ContractPeriod"].ToString();
                //TxtJobDet.Text = dt.Tables[0].Rows[0]["JobDetail"].ToString();
                TxtFulTMPTM.Text = dt.Tables[0].Rows[0]["FT_PTpayment"].ToString();
                TextPIEmail.Text = dt.Tables[0].Rows[0]["PI_EmailID"].ToString();
                int iddep = Convert.ToInt32(dt.Tables[0].Rows[0]["Department_idDepartment"].ToString());
                PositionCombo(iddep, DDLDepartment);
                string emptype = dt.Tables[0].Rows[0]["emptype_idEmpType"].ToString();
                string dtt = dt.Tables[0].Rows[0]["sendforapproval"].ToString();
                //if (dt.Tables[0].Rows[0]["sendforapproval"].ToString() == "0")
                //    ChkSendApproval.Checked = false;
                //else
                //    ChkSendApproval.Checked = true;
                int mpos = 0;
                foreach (ListItem itm in DDLEmpType.Items)
                {
                    if (itm.Value == emptype)
                        DDLEmpType.SelectedIndex = mpos;
                    mpos++;
                }

                GetProjects();
                for (int m = 0; m < DDLSourceFund.Items.Count; m++)
                {
                    if (DDLSourceFund.Items[m].Value == dt.Tables[0].Rows[0]["SourceofFund"].ToString())
                    {
                        DDLSourceFund.SelectedIndex = m;
                        break;
                    }
                }
                if (dt.Tables[0].Rows[0]["OnReq"].ToString() == "2")
                {
                    RDOOnReq1.Checked = true;
                    RDOOnReq2.Checked = false;
                }
                else if (dt.Tables[0].Rows[0]["OnReq"].ToString() == "3")
                {
                    RDOOnReq2.Checked = true;
                    RDOOnReq1.Checked = false;
                }
                else
                {
                    RDOOnReq2.Checked = false;
                    RDOOnReq1.Checked = false;
                    RDOOnReq3.Checked = true;
                    TxtOnReqTxt.Text = dt.Tables[0].Rows[0]["OnReq"].ToString();
                }
                int sFund =Convert.ToInt32( dt.Tables[0].Rows[0]["SourceofFund"].ToString());
                RAS.GrantWebData.WebGrantSoapClient mydtx = new RAS.GrantWebData.WebGrantSoapClient();
                LblSourceFund.Text = mydtx.GetProjectString(sFund.ToString());
                GetFunSourceDetail("'" + DDLSourceFund.Items[0].Text + "'");

                if (dt.Tables[0].Rows[0]["sendforapproval"].ToString() == "0")
                {
                    Btn_SaveApp.Enabled = true;
                    

                //    Btn_PrintAttach.Enabled = true;
                //    Btn_PrintForm.Enabled = true; 
                }
                else
                {
                    if (grv.Cells[5].Text.Equals("Returned") == false)
                    {
                        Btn_SaveApp.Enabled = false;
                    }
                   
                }
                //SetVFundsource(false);
                

            }
        }

        private void BindjobDetail(string id)
        {
            try
            {
                DataSet dt = new DataSet();
                dt = mySQLConnection.getmySQLdataset("Select * from job_details where idapplication =" + id);
                if (dt.Tables[0].Rows.Count > 0)
                {
                    for (int mlp =1; mlp < 5; mlp++)
                    {
                        TextBox mytext = (TextBox)PnlNewApp.FindControl("TxtJobTask" + mlp);
                        mytext.Text = dt.Tables[0].Rows[mlp-1]["JobTask"].ToString();
                        string jobid = dt.Tables[0].Rows[mlp - 1]["idJob_Details"].ToString();
                        if (mlp == 1) H1.Value = jobid;
                        if (mlp == 2) H2.Value = jobid;
                        if (mlp == 3) H3.Value = jobid;
                        if (mlp == 4) H4.Value = jobid;
                    }
                }
            }
            catch (Exception ex)
            { 
            }
        }

        //private void SetVFundsource(bool state)
        //{
        //    DDLFundSrc.Visible = state;
        //    DDLFundSrc0.Visible = state;
        //    DDLFundSrc1.Visible = state;
        //    DDLFundSrc2.Visible = state;
        //    DDLFundSrc3.Visible = state;
        //}
        //protected void Btn_PrintForm_Click(object sender, EventArgs e)
        //{
           

            
        //    //AttachPDF(Path.Combine(Server.MapPath(" ") + "\\" + "CForm\\F12A.pdf"), Path.Combine(Server.MapPath(" ") + "\\" + "imagesDir\\" + Session["UID"].ToString()) + "\\temp.pdf", Path.Combine(Server.MapPath(" ") + "\\" + "imagesDir\\" + Session["UID"].ToString()));
        //}
       
        protected void BtnUploadFiles_Click(object sender, EventArgs e)
        {
            try
            {
                if (FileUpload1.HasFile)
                {
                    string myid = Guid.NewGuid().ToString();

                    string ftype = FileUpload1.FileName.ToString();
                    ftype = ftype.Substring(ftype.IndexOf(".", ftype.Length - 5));
                    string dirpath = Path.Combine(Server.MapPath(" ") + "\\" + "imagesDir\\" + Session["UID"].ToString());
                    //dirpath = Regex.Replace(dirpath, "mobi", "");
                    Directory.CreateDirectory(dirpath);
                    if (RadioButton1.Checked == true) myid = "SelfImage";
                    if (RadioButton2.Checked == true) myid = "Passport01";
                    if (RadioButton3.Checked == true) myid = "Passport02";
                    string[] files = Directory.GetFiles(dirpath,myid + "*.*" );
                    foreach (string itm in files)
                    {
                        File.Delete(itm);
                    }
                    
                    
                        
                    
                    myid = myid + DateTime.Now.ToString("HH-MM-ss");
                    string fullPath = Path.Combine(dirpath + "\\" + myid + ftype);
                    
                    FileUpload1.SaveAs(fullPath);
                    Image1.ImageUrl = "~/imagesDir/" + Session["UID"].ToString() + "/" + myid + ftype;
                }
            }
            catch (Exception ex)
            {
                string tt = ex.ToString();
                
            }
        }

        protected void RadioButton1_CheckedChanged(object sender, EventArgs e)
        {
            ShowImage();
        }

        private void ShowImage()
        {

            try
            {
                string myid = "";
                if (RadioButton1.Checked == true) myid = "SelfImage";
                if (RadioButton2.Checked == true) myid = "Passport01";
                if (RadioButton3.Checked == true) myid = "Passport02";
                if (Directory.Exists(Server.MapPath(" ") + "\\" + "imagesDir\\" + Session["UID"].ToString()) == false)
                {
                    Directory.CreateDirectory(Server.MapPath(" ") + "\\" + "imagesDir\\" + Session["UID"].ToString());
                }
                string dirpath = Path.Combine(Server.MapPath(" ") + "\\" + "imagesDir\\" + Session["UID"].ToString());
                string[] files = Directory.GetFiles(dirpath, myid + "*.*");
                if (files.Length == 0)
                {
                    Image1.ImageUrl = "~/image/Inotfound.jpg" ;
                    return;
                }
                string filename = Path.GetFileName(files[0]);
                Image1.ImageUrl = "~/imagesDir/" + Session["UID"].ToString() + "/" + filename;

                string URL = Page.ResolveClientUrl(Image1.ImageUrl);
                Image1.OnClientClick = "window.open('" + URL + "'); return false;";
            }
            catch (Exception ex)
            {
                mySQLConnection.ErrorReport(ex.ToString());
                
            }
            
        }

        protected void RadioButton2_CheckedChanged(object sender, EventArgs e)
        {
            ShowImage();
        }

        protected void RadioButton3_CheckedChanged(object sender, EventArgs e)
        {
            ShowImage();
        }

        protected void Image1_Click(object sender, ImageClickEventArgs e)
        {

            
            
        }

        //protected void DDLFundSrc_SelectedIndexChanged(object sender, EventArgs e)
        //{
        //    FillFundSource(DDLFundSrc0, 1, DDLFundSrc.SelectedItem.Text);
        //    FillFundSource(DDLFundSrc1, 2, DDLFundSrc.SelectedItem.Text + "/" + DDLFundSrc0.SelectedItem.Text);
        //    FillFundSource(DDLFundSrc2, 3, DDLFundSrc.SelectedItem.Text + "/" + DDLFundSrc0.SelectedItem.Text + "/" + DDLFundSrc1.SelectedItem.Text);
        //    FillFundSource(DDLFundSrc3, 4, DDLFundSrc.SelectedItem.Text + "/" + DDLFundSrc0.SelectedItem.Text + "/" + DDLFundSrc1.SelectedItem.Text + "/" + DDLFundSrc2.SelectedItem.Text);

        //    GetProjectName();
        //}

        //private void GetProjectName()
        //{
        //    try
        //    {
        //        RAS.GrantWebData.WebGrantSoapClient mygr = new RAS.GrantWebData.WebGrantSoapClient();
        //        string dt = GetProjectComboInfo();
        //        string[] PIInfo = Regex.Split(mygr.GetPIDetail("'" + dt + "'"), "####");
        //        LBLFundDetail.Text = PIInfo[2];
        //    }
        //    catch (Exception)
        //    {}
        //}

        //protected void DDLFundSrc0_SelectedIndexChanged(object sender, EventArgs e)
        //{
        //    FillFundSource(DDLFundSrc1, 2,DDLFundSrc.SelectedItem.Text + "/"+ DDLFundSrc0.SelectedItem.Text);
        //    FillFundSource(DDLFundSrc2, 3, DDLFundSrc.SelectedItem.Text + "/" + DDLFundSrc0.SelectedItem.Text + "/" + DDLFundSrc1.SelectedItem.Text);
        //    FillFundSource(DDLFundSrc3, 4, DDLFundSrc.SelectedItem.Text + "/" + DDLFundSrc0.SelectedItem.Text + "/" + DDLFundSrc1.SelectedItem.Text + "/" + DDLFundSrc2.SelectedItem.Text);
        //    GetProjectName();
        //}

        //protected void DDLFundSrc1_SelectedIndexChanged(object sender, EventArgs e)
        //{
        //    FillFundSource(DDLFundSrc2, 3,DDLFundSrc.SelectedItem.Text + "/"+ DDLFundSrc0.SelectedItem.Text + "/" + DDLFundSrc1.SelectedItem.Text);
        //    FillFundSource(DDLFundSrc3, 4, DDLFundSrc.SelectedItem.Text + "/" + DDLFundSrc0.SelectedItem.Text + "/" + DDLFundSrc1.SelectedItem.Text + "/" + DDLFundSrc2.SelectedItem.Text);
        //    GetProjectName();
        //}

        //protected void DDLFundSrc2_SelectedIndexChanged(object sender, EventArgs e)
        //{
        //    FillFundSource(DDLFundSrc3, 4, DDLFundSrc.SelectedItem.Text + "/" + DDLFundSrc0.SelectedItem.Text + "/" + DDLFundSrc1.SelectedItem.Text + "/" + DDLFundSrc2.SelectedItem.Text);
        //    GetProjectName();
           
        //}

        //protected void DDLFundSrc3_SelectedIndexChanged(object sender, EventArgs e)
        //{
        //    GetProjectName();
        //    //FillFundSource(DDLFundSrc4, 5, DDLFundSrc.SelectedItem.Text + "/" + DDLFundSrc0.SelectedItem.Text + "/" + DDLFundSrc1.SelectedItem.Text + "/" + DDLFundSrc2.SelectedItem.Text + "/" + DDLFundSrc3.SelectedItem.Text);
        //}

        protected void GridView1_DataBound(object sender, EventArgs e)
        {
 
        }

        protected void GridView1_RowDataBound(object sender, GridViewRowEventArgs e)
        {
            
                if (e.Row.RowType == DataControlRowType.DataRow)
                {
                     switch (e.Row.Cells[5].Text)
                        {
                              case "Returned":
                                    e.Row.BackColor =System.Drawing.Color.Tomato;
                                    //e.Row.BackColor = Color.Green;
                                    break;
                              case "Applied":
                                    e.Row.BackColor = System.Drawing.Color.LightGray;
                                    //e.Row.BackColor = Color.Blue;
                                    break;
                              case "Cancelled":
                                    e.Row.BackColor = System.Drawing.Color.IndianRed;
                                    //e.Row.BackColor = Color.Black;
                                    break;
                              case "Approved":
                                    e.Row.BackColor = System.Drawing.Color.LawnGreen;
                                    //e.Row.BackColor = Color.Black;
                                    break;
                         case "Not Applied":
                                    e.Row.BackColor = System.Drawing.Color.Yellow;
                                    break;

                             
                        }
                }
             }

        protected void BtnSrchFund_Click(object sender, ImageClickEventArgs e)
        {
            if (TextPIEmail.Text == "")
            {
                ShowPopUpMsg("Please enter the PI name.");
                return;
            }
            DDLSourceFund.Items.Clear();
            if (TextPIEmail.Text.IndexOf("@") < 0)
                TextPIEmail.Text = TextPIEmail.Text + "@squ.edu.om";
            GetProjects();
            if(DDLSourceFund.Items.Count>0)
            GetFunSourceDetail("'" + DDLSourceFund.Items[0].Text+ "'");
        }

        private void GetProjects()
        {
            RAS.GrantWebData.WebGrantSoapClient mydtx = new RAS.GrantWebData.WebGrantSoapClient();
            List<string> dta = mydtx.GetGrantsbyEmail(TextPIEmail.Text);
            if (dta.Count == 0)
                ShowPopUpMsg("No record found.");
            foreach (string itm in dta)
            {
                ListItem myitm = new ListItem();
                string[] dt = Regex.Split(itm, "#");
                myitm.Value = dt[0];
                myitm.Text = dt[1];
                DDLSourceFund.Items.Add(myitm);
            }
        }

        protected void DDLSourceFund_SelectedIndexChanged(object sender, EventArgs e)
        {
            GetFunSourceDetail("'" + DDLSourceFund.SelectedItem.Text + "'");
        }

        private void GetFunSourceDetail(string dtax)
        {
            RAS.GrantWebData.WebGrantSoapClient mydtx = new RAS.GrantWebData.WebGrantSoapClient();
            string[] dta = Regex.Split(mydtx.GetPIDetail(dtax), "####");
            LBLFundDetail.Text = dta[2];
        }

        protected void ChkOnReq1_CheckedChanged(object sender, EventArgs e)
        {
            if (RDOOnReq1.Checked == true) TxtFulTMPTM.Text = "";
        }

        protected void ChkOnReq2_CheckedChanged(object sender, EventArgs e)
        {
            if (RDOOnReq2.Checked == true) TxtFulTMPTM.Text = "";
        }

        protected void TxtOnReqTxt_TextChanged(object sender, EventArgs e)
        {
            TxtFulTMPTM.Text = "";
        }

        protected void TxtFulTMPTM_TextChanged(object sender, EventArgs e)
        {
            TxtOnReqTxt.Text = "";
        }

        protected void DDLEmpType_SelectedIndexChanged(object sender, EventArgs e)
        {
            paymentoptionEnDis();
        }

        private void paymentoptionEnDis()
        {
            if (DDLEmpType.SelectedItem.Text.Equals("Full Time", StringComparison.InvariantCultureIgnoreCase) == true)
            {
                Panel1.Enabled = false;
                TxtFulTMPTM.Enabled = true;
            }
            else
            {
                TxtFulTMPTM.Enabled = false;
                Panel1.Enabled = true;
            }
        }

        protected void RDOOnReq3_CheckedChanged(object sender, EventArgs e)
        {
            if (RDOOnReq3.Checked == true) TxtOnReqTxt.Enabled = true;
            else
            {
                TxtOnReqTxt.Enabled = false;
                TxtOnReqTxt.Text = "";
            }

        }

        protected void DtaWorkPlc_Selected(object sender, EntityDataSourceSelectedEventArgs e)
        {
            //if (Session["WKLOC"]!= null)
            //{
            //    int wrk =Convert.ToInt32( Session["WKLOC"].ToString());
            //    PositionCombo(wrk, DDLWorkArea);
            //}
        }

       

        //protected void Btn_PrintAttach_Click(object sender, EventArgs e)
        //{
            
        //}

        //protected void BtnAddJobDetail_Click(object sender, EventArgs e)
        //{
        //    DataTable dt = GrdJobDetail.DataSource as DataTable;
        //    if (dt != null)
        //    {
        //        DataRow dr = dt.NewRow();
        //        dt.Rows.Add(dr);
        //        dt.AcceptChanges();
                
        //    }
        //}

        protected void DetailsView1_ItemInserting(object sender, DetailsViewInsertEventArgs e)
        {
            //var et = (e.Entity as bankbranch);
            //et.BankMasterIDBankMast = Convert.ToInt32(DDLBankName.SelectedValue);
        }

        //protected void DtaJobDetails_Inserting(object sender, EntityDataSourceChangingEventArgs e)
        //{
        //    job_details jd = (job_details)e.Entity;
        //    if (jd.JobTask == null) return;
        //    if (GridView1.SelectedRow == null) return;
        //    if (GrdJobDetail.Rows.Count >= 4)
        //    {
        //        e.Cancel = true;
        //        ShowPopUpMsg("More than four items are not allowed.");
        //        return;
        //    }

        //    GridViewRow grv = GridView1.SelectedRow;
        //    string id = grv.Cells[2].Text;

        //    var et = (e.Entity as job_details );
        //    et.idApplication = Convert.ToInt32(id);

        //    BindjobDetail(id);
            
        //}

        protected void GrdJobDetail_SelectedIndexChanged(object sender, EventArgs e)
        {
            //DetailsView1.PageIndex = GrdJobDetail.SelectedIndex;
            //DetailsView1.ChangeMode(DetailsViewMode.ReadOnly);
        }

        protected void DDLDept_SelectedIndexChanged(object sender, EventArgs e)
        {
            string t = DDLDepartment.SelectedValue;
        }

        protected void GridView1_PageIndexChanging(object sender, GridViewPageEventArgs e)
        {
            GridView1.AllowPaging = true;
            GridView1.PageIndex = e.NewPageIndex;
            BindGridData();
        }

        protected void Btn_SaveFinal_Click(object sender, EventArgs e)
        {
            SaveApp(true);
        }

        protected void Btn_SaveFinalApproval_Click(object sender, EventArgs e)
        {
           
        }

        protected void DDLEmpStatus_SelectedIndexChanged(object sender, EventArgs e)
        {
            DisableStaff();
        }

        private void DisableStaff()
        {
            if (DDLEmpStatus.SelectedItem.Text.Equals("SQU Staff", StringComparison.InvariantCultureIgnoreCase) == true)
            {
                TxtEmpNo.Enabled = true;
            }
            else
            {
                TxtEmpNo.Enabled = false;
                TxtEmpNo.Text = "";
            }
        }

        protected void Btn_PrintForm_Click(object sender, ImageClickEventArgs e)
        {
            ImageDirtoPDF("temp.pdf", Path.Combine(Server.MapPath(" ") + "\\" + "imagesDir\\" + Session["UID"].ToString()));
            WriteFile("F12A.pdf");
        }

        protected void Btn_PrintAttach_Click(object sender, ImageClickEventArgs e)
        {
            try
            {
                int pgval = ImageDirtoPDF("temp.pdf", Path.Combine(Server.MapPath(" ") + "\\" + "imagesDir\\" + Session["UID"].ToString()));
                if (pgval == 0)
                {
                    ShowPopUpMsg("No accachment found.");
                    return;
                }
                Response.ContentType = "application/octetstream";
                Response.AppendHeader("Content-Disposition", "attachment; filename=temp.pdf");

                string fname = Path.Combine(Server.MapPath(" ") + "\\" + "imagesDir\\" + Session["UID"].ToString() + "\\temp.pdf");
                PdfReader pdfreader = new PdfReader(Path.Combine(Server.MapPath(" ") + "\\" + "imagesDir\\" + Session["UID"].ToString() + "\\temp.pdf"), null);
                using (PdfStamper ps = new PdfStamper(pdfreader, Response.OutputStream))
                {
                }
                Response.End();
                pdfreader.Close();
            }
            catch (Exception)
            {}
        }

        protected void Btn_SaveApp_Click1(object sender, EventArgs e)
        {
            SaveApp(false);
        }
        }
}
