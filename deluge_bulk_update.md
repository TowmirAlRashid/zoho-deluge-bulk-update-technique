// let's say, you are going to push some data from the contacts to the deals in a ZOHO CRM. If the records quantity is very hign, it can be costly in time. So there is a easier method for this:

// Get the data from all contacts like this way:
start_page = 1;
per_round = 10;
per_page = 200;
module_api_name = "Contacts";
loop_list = ".".leftPad(per_round).toList("");
page_no = (start_page - 1) * per_round;
upd_map = Map();
for each  abc in loop_list
{
	page_no = page_no + 1;
	records = zoho.crm.getRecords(module_api_name,page_no,per_page);
	for each  record in records
	{
		upd_map.put(record.get("id") + "",{"Mailing_Street":ifnull(record.get("Mailing_Street"),""),"Mailing_City":ifnull(record.get("Mailing_City"),""),"Mailing_Province":ifnull(record.get("Mailing_State"),""),
"Mailing_Postcode":ifnull(record.get("Mailing_Zip"),""),"Email":ifnull(record.get("Email"),""),"Phone":ifnull(record.get("Phone"),""),"Mobile":ifnull(record.get("Mobile"),"")});
	}
}
info upd_map;

// create another map for contId:dealId mapping:
start_page = 5;
per_round = 10;
per_page = 200;
module_api_name = "Deals";
loop_list = ".".leftPad(per_round).toList("");
page_no = (start_page - 1) * per_round;
upd_map = Map();
for each  abc in loop_list
{
	page_no = page_no + 1;
	records = zoho.crm.getRecords(module_api_name,page_no,per_page);
	for each  record in records
	{
		if(record.get("Contact_Name") != null)
		{
			upd_map.put(record.get("Contact_Name").get("id") + "",record.get("id"));
		}
	}
}
info upd_map;

// after this, the data from those 2 module runs, should be kept in 2 different JSON files in a folder, you can use VSCODE for this.
// in the same folder, create an app.js file and use this:
const fs = require("fs");
const contact_map = require("./contact_map.json");
const claim_map = require("./claim_map.json");

let final_json_list = [];

Object.keys(claim_map).forEach((cont_id) => {
  let indvMap = { id: claim_map[cont_id], ...contact_map[cont_id] };
  final_json_list.push(indvMap);
});
fs.writeFile(
  "final_json_list.json",
  JSON.stringify(final_json_list),
  function (err) {
    if (err) {
      console.log(err);
    }
  }
);

// this will generate an output file that you can use to bulk update your deals using the code below:
claim_list = {};
upd_list = list();
for each  claim in claim_list
{
	upd_list.add(claim);
	if(upd_list.size() == 100)
	{
		zoho.crm.bulkUpdate("Deals",upd_list,{"trigger":{""}});
		upd_list = list();
	}
}
if(upd_list.size() > 0)
{
	zoho.crm.bulkUpdate("Deals",upd_list,{"trigger":{""}});
	upd_list = list();
}
