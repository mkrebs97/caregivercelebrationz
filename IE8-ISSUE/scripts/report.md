// for IEprompt call back
function promptCallback(val, elmt){
	if(elmt.length > 0){
		elmt.attr("title", val);
		var id = elmt.attr("id").substring(8);
		var noteInput = $("#note"+id);
		noteInput.val(val);
		console.log(id, noteInput);
		if (noteInput.val() == ""){
			elmt.html("Add Note");
		}else{
			elmt.html("Edit Note");
		} 
	}
}
$(function(){
	/* orginal function for download
function startDownload(page)
	{
		var rndId = new Date().getTime();
		document.location.href = "downloadcsv.cfm?page=" + page + "&temp=#url.temp#&start_date=#start_date#&end_date=#end_date##extraParams#&id=" + rndId;
		$("##downloadCSVLink").slideUp();
		$("##waitForDownload").slideDown();
		setTimeout("timer(" + rndId + ")", 1000);
	}
*/
	/*
	download button interactions
	*/
	var cometId ="";
	function timer(rndId)
	{
		$.post("comet.cfm?id=" + rndId, function(data){
				cometId = data;
			});
		if ($.trim(cometId).substring(0, 1) == "1")
		{
			$("#waitForDownload").slideUp();
			$("#downloadCSVLink").slideDown();
		}
		else
		{
			setTimeout(function(){timer(rndId);}, 1000);
		}
	}
		/*
	deleted line 56 COSOLE call for production - caused errors-mistakenly left in
	*/
	function downloadCsv(url){
		var rndId = new Date().getTime();
		$("#downloadCSVLink").slideUp();
		$("#waitForDownload").slideDown();
		setTimeout(function(){timer(rndId);}, 1000);
		document.location.href = url+"&id=" + rndId;
	}
	$("#downloadCSVLink").on("click", "a", function(e){
		e.preventDefault();
		var csvUrl = $(this).attr("href");
		downloadCsv(csvUrl);
	});
	/*
	autocomplete sender and recipient
	*/
	var ajaxUrl = "/rmi/ajax.cfm?temp="+urlParams.temp;
	var searchData = {page: "report", action: "search",};
	function search(url, urlData, inputId, resultId){
		$("#"+inputId).autocomplete({
			source: function(request, response){
									urlData.q = request.term;
									// remember to set showDebugOutput = false in ajax.cfm
									$.get(url, urlData, function(data){
													response(data);
												}, 'json');
								},
				minLength: 2,
				select: function(event, ui){
					$("#"+resultId).val(ui.item.value);
					$("#"+inputId).val(ui.item.label);
					return false;
				}
		});
	}
	if(urlParams.page == "budget" && urlParams['function'] == "transfer"){
		var toData = jQuery.extend(true, {}, searchData);
		toData.budget = "true";
		toData.manager = "yes";
		var fromData = jQuery.extend(true, {}, searchData);
		fromData.editbudget = "yes";
		fromData.frombudget = "yes";
		fromData.showStatus = "no";
		if(typeof includeInstitue != "undefined"){
			toData.institute = includeInstitue;
			fromData.institute = includeInstitue;
		}
		search(ajaxUrl, toData, "toID", "toemployees");
		search(ajaxUrl, fromData, "fromID", "fromemployees");
	}else{
		search(ajaxUrl, searchData, "toID", "toemployees");
		search(ajaxUrl, searchData, "fromID", "fromemployees");
		// budget 
		search(ajaxUrl, searchData, "manager_auto_complete", "managerid");
	}
	/*
$("#toID").autocomplete({
		source: function(request, response){
								var url = ajaxUrl;
								var urlData = {	page: "report",
																action: "search", 
																q: request.term};
								// remember to set showDebugOutput = false in ajax.cfm
								$.get(url, urlData, function(data){
												response(data);
											}, 'json');
							},
			minLength: 2,
			select: function(event, ui){
				$("#toemployees").val(ui.item.value);
				$("#toID").val(ui.item.label);
				return false;
			}
	});

	$("#fromID").autocomplete({
		source: function(request, response){
								var url = ajaxUrl;
								var urlData = {	page: "report",
																action: "search", 
																q: request.term};
								// remember to set showDebugOutput = false in ajax.cfm
								$.get(url, urlData, function(data){
												response(data);
											}, 'json');
							},
			minLength: 2,
			select: function(event, ui){
				$("#fromemployees").val(ui.item.value);
				$("#fromID").val(ui.item.label);
				return false;
			}
	});
	
	$("#manager_auto_complete").autocomplete({
		source: function(request, response){
								var url = ajaxUrl;
								var urlData = {	page: "report",
																action: "search", 
																q: request.term};
								// remember to set showDebugOutput = false in ajax.cfm
								$.get(url, urlData, function(data){
												response(data);
											}, 'json');
							},
			minLength: 2,
			select: function(event, ui){
				$("#managerid").val(ui.item.value);
				$("#manager_auto_complete").val(ui.item.label);
				return false;
			}
	});
	*/
	$(".budget-change-form").submit(function(e){
		var amountEl = $(".amount", this);
		var amount = amountEl.val();
		var actionEl = $(".action", this);
		var action = actionEl.val();
		var budget = $.trim($(this).closest("tr").find(".budget-available").text());
		budget = Number(budget.replace(/[^0-9\.]+/g,""));
		if(amount == "" || isNaN(amount) || amount <= 0){
			alert("The Amount must consist of positive numbers only.");
			amountEl.focus();
			return false;
		}
		if(action == "-" && budget < amount){
			alert("The AMOUNT cannot be less than the available budget.");
			console.log(amountEl);
			amountEl.focus();
			return false;
		}
		console.log("form will be submitted "+action+" "+amount);
		return true;
	});
	
	$("a.notelink").on("click", function(e){
		e.preventDefault();
		var text = $.trim($(this).attr("title"));
		console.log(text);
		IEprompt("Please enter the note for this transaction:", text, $(this));
	});
	$("table").tooltip({
		element: ".notelink",
		content: function(){
			var text = $.trim($(this).attr("title"));
			if(text != ""){
				return "<div class='aobject'>Transaction Note</div><div class='content'>"+text+"</div>";
			}
		}
	});
	/*
	ajax to refresh data table
	*/
	/*
$("#filter-form").submit(function(e){
		e.preventDefault();
		formData = $(this).serialize();
		console.log(myConfigs);
	});
	function refreshDataTable(url, elementId, myColumnDefs, myDataSource, myConfigs){
		var myDataTable = new YAHOO.widget.DataTable(elementId, myColumnDefs, myDataSource, myConfigs);
				// Update totalRecords on the fly with value from server
				myDataTable.handleDataReturnPayload = function(oRequest, oResponse, oPayload) {
					oPayload.totalRecords = oResponse.meta.totalRecords;
					if (oResponse.meta.totalRecords > 0)
					{
						$("#reportRowText").html("a " + oResponse.meta.totalRecords + " row ");
					}
					return oPayload;
				}

				return {
					ds: myDataSource,
					dt: myDataTable
				};
	}
*/
	// notecard report
	$("#sender-autocomplete").autocomplete({
		source: function(request, response){
								var url = ajaxUrl;
								var urlData = {	page: "award",
																division: "lookup",
																action: "search", 
																q: request.term};
								// remember to set showDebugOutput = false in ajax.cfm
								$.get(url, urlData, function(data){
												response(data);
											}, 'json');
							},
			minLength: 2,
			select: function(event, ui){
				$("#managerid").val(ui.item.value);
				var name = ""+ui.item.label.split("-", 1);
				$(this).parent().find("#fname").val($.trim(name.split(",")[1]));
				$(this).parent().find("#lname").val($.trim(name.split(",")[0]));
				$("#sender-autocomplete").val(ui.item.label);
				return false;
			}
	});
	$(".yui-dt table").on("click", ".yui-dt-col-date a", function(e){
		e.preventDefault();
		// ie8 doesnt allow title to has spaces, dashes, or other punctuation
		popupwindow(this.href, "viewing_award", 600, 550);
	});
	function popupwindow(url, title, w, h) {
	  var left = (screen.width/2)-(w/2);
	  var top = (screen.height/2)-(h/2);
		var specs = 'toolbar=no, location=no, directories=no, status=no, menubar=no,';
		specs += 'scrollbars=no, resizable=no, copyhistory=no, width='+w+', height='+h;
		specs += ', top='+top+', left='+left;
	  return window.open(url, title, specs);
	
	} 
	
});
