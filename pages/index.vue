<template>
<div>
  <div class="container">
    <div>
      <logo />
      <h5 class="title">XML_DATA</h5>     
    </div>   
    <iframe src ="https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi?retmode=json&retmax=10&term=((medical" />
    <iframe src ="https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi?retmode=json&retmax=10&term=((medical%20university%20of%20South%20Carolina[Affiliation])%20AND%20orcid[auid])" />
  </div>
  <div>
     <iframe v-if="flag" src="https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=pubmed&retmode=xml&id=32034076,32033032,32032440,32025343,32024424,32023372,32022559"  width="100%" height="350px"></iframe>
      <nuxt-link :to="{ path: '/page', query: { id: 'Page1'}}">
        <h2 class="subtitle">My Page</h2>
      </nuxt-link>
      <button @click="flag = ! flag">Search</button>
    </div>
</div>
</template>


<script>
import Logo from "~/components/Logo.vue";

export default {
  
  components: {
    Logo
  },
  data:function(){
    return{
      flag:false
    }
  },
  methods: {
    searchClick: function() {
      // var newArray = [];
      // for (var i = 0; i < this.pmidList.length; i++) {
      //   newArray.push(this.pmidList[i]["PMID"]);
      // }
      this.dialogLoader = true;
      // let newArray = this.pmidList.map(a => a.PMID);

      let newArray = this.articles.map(a => a.PMID);

      var self = this;
      self.overlay_name = true;

      //   self.articles = [];
      // self.searchedPubs = [];
      let dbid = this.$store.getters.dbid;

      //   this.selectedRecords.forEach(function(arg, index) {

      let memberID = this.nameData["MemberID"];
      let firstName = this.nameData["First_Name"];
      let MI = this.nameData["Middle_Initial"];
      let lastName = this.nameData["Last_Name"];
      let suffix = ""; // $("suffix").val();
      let affiliation = "";
      let affiliations = [];
      if (this.nameData["Affiliations"].includes(";")) {
        affiliations = this.nameData["Affiliations"].split(";");
        affiliation = affiliations[0];
      } else {
        affiliation = this.nameData["Affiliations"];
      }

      let yearStart = this.nameData["Year_Start"];
      let yearTo = this.nameData["Year_End"];
      let Search_String = this.nameData["Search_String"];
      let pmidList = this.nameData.Pending;
      let fromYear = this.nameData["Year_Start"];
      let toYear = "3000";
      let new_res = [];

      this.nameData["Last_Name"] +
        " " +
        this.nameData["First_Name"].charAt(0) +
        this.nameData["Middle_Initial"].charAt(0);

      let currentTerm = "";

      if (affiliations.length == 0) {
        currentTerm =
          "((" +
          Strings.orEmpty(lastName) +
          " " +
          Strings.orEmpty(firstName).substring(0, 1) +
          Strings.orEmpty(MI).substring(0, 1) +
          "[AU]) AND " +
          Strings.orEmpty(affiliation) +
          "[AD]) AND (" +
          '"' +
          fromYear +
          '"' +
          "[DP]  : " +
          '"' +
          toYear +
          '"' +
          "[DP])";
      } else {
        currentTerm =
          "((" +
          Strings.orEmpty(lastName) +
          " " +
          Strings.orEmpty(firstName).substring(0, 1) +
          Strings.orEmpty(MI).substring(0, 1) +
          "[AU]) AND " +
          Strings.orEmpty(affiliations[0]) +
          "[AD]) AND (" +
          '"' +
          fromYear +
          '"' +
          "[DP]  : " +
          '"' +
          toYear +
          '"' +
          "[DP])";
      }

      if (Search_String.length > 0) {
        currentTerm = Search_String;
      }
      axios
        .get(
          "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi?tool=biosketch&email=morgenww@musc.edu&db=pubmed&api_key=d0784f454c4ca9a3cb738d33f97b4ae34808&retmode=json&retmax=500&term=" +
            currentTerm,
          {}
        )
        .then(res => {
          self.dialogLoader = false;
          var obj = res.data.esearchresult.idlist;
          var p = obj.join(",");
          //  console.log(p);
          if (typeof pmidList != "undefined") {
            //const arr2 = pmidList.split(",");
            // const arr2 = newArray.split(",");
            const arr2 = newArray;
            new_res = obj.filter(v => !arr2.includes(v));
          }
          self.pmidListPubMed = new_res.join(",");
          this.pmidInput = "Search term: \n" + currentTerm + " " + "\n" + "\n";
          if (new_res.length > 0) {
            this.pmidInput +=
              "Search found: " +
              "\n" +
              "\n" +
              p +
              "\n" +
              "\n" +
              "New Publications found:" +
              "\n" +
              "\n" +
              new_res;
          } else {
            this.pmidInput +=
              "Search found: " +
              "\n" +
              "\n" +
              p +
              "\n" +
              "\n" +
              " publications; however, none were new.";
          }
          self.overlay_name = false;
        })

        .catch(function(error) {
          self.overlay_name = false;
          //  console.log(error);
        });
    },
    pubMedXML(pmid) {
      let authorDataTMP = [];
      this.authorData = [];
      this.abstract = "";
      var self = this;
      var text, parser, xmlDoc;
      //  this.overlay_name = true;
      var start = Date.now();
      axios
        .get(
          "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=pubmed&retmode=xml&id=" +
            pmid +
            "&api_key=d0784f454c4ca9a3cb738d33f97b4ae34808&tool=biosketch_program&email=morgenww@musc.edu"
        )
        .then(response => {
          this.overlay_name = false;

          parser = new DOMParser();
          xmlDoc = parser.parseFromString(response.data, "text/xml");
          const abstracts = xmlDoc.querySelectorAll("AbstractText");
          let abstract_text = "";
          abstracts.forEach(a => {
            if (a.getAttribute("Label") !== null) {
              abstract_text +=
                "<b>" + a.getAttribute("Label") + "</b>: " + a.textContent;
            } else {
              abstract_text += a.textContent;
            }
            abstract_text += "<br /><br />";
          });
          this.abstract = abstract_text;

          const AuthorsInfo = xmlDoc.querySelectorAll("Author");
          let count = 1;
          AuthorsInfo.forEach(a => {
            let itemsAdd = {};
            let affiliations = "";
            let AuthorName = "";
            itemsAdd["memberID"] = 0;
            itemsAdd["PmPubsAuthorID"] = 0;
            itemsAdd["pmid"] = "";
            itemsAdd["Program_Code"] = "";
            itemsAdd["checked"] = false;
            itemsAdd["ProgramCode"] = "";
            itemsAdd["emphasis"] = "";
            itemsAdd["Color"] = "";

            for (var i = 0; i < a.children.length; i++) {
              let tt = a.children[i].nodeName;
              let t = a.children[i].textContent;

              if (tt === "AffiliationInfo") {
                affiliations +=
                  "<b>Affiliation:</b>" + a.children[i].textContent;
                affiliations += "<br /><br />";
                //    itemsAdd["Affiliation"] = a.children[i].textContent;
                itemsAdd["Affiliation"] = affiliations;
              } else if (tt === "LastName") {
                itemsAdd["Last_Name"] = a.children[i].textContent;
                AuthorName = a.children[i].textContent;

                //people.filter(p => p.dinner == "sushi")

                let result = self.memberNames.filter(p => p.LastName == t);
                console.log(result);

                // if (this.nameData["Last_Name"] === a.children[i].textContent) {
                if (result.length > 0) {
                  itemsAdd["checked"] = true;
                  itemsAdd["memberID"] = result[0]["memberID"];
                  itemsAdd["ProgramCode"] = result[0]["Program_Code"];
                  itemsAdd["Program_Code"] = result[0]["Program_Code"];
                  //result["Program_Code"];
                  itemsAdd["Color"] = result[0]["Color"];
                  itemsAdd["memberID"] = result[0]["memberID"];

                  //   itemsAdd["checked"] = true;
                  //   itemsAdd["ProgramCode"] = this.nameData["Program_Code"];
                  //   itemsAdd["Program_Code"] = this.nameData["Program_Code"];
                  //   itemsAdd["Color"] = this.nameData["Color"];
                }
              } else if (tt === "Initials") {
                itemsAdd["Initials"] = a.children[i].textContent;
                AuthorName += " " + a.children[i].textContent;
              } else if (tt === "ForeName") {
                itemsAdd["ForeName"] = a.children[i].textContent;
                AuthorName += " " + a.children[i].textContent;
              }
            }
            count += 1;

            itemsAdd["PmPubsAuthorID"] = count;

            authorDataTMP.push(itemsAdd);

            //  alert(AuthorName + " " + affiliations);
          });
          // alert("done");

          this.authorData = authorDataTMP;

          var end = Date.now();
          var diff = end - start;

          this.networkSpeed = (diff / 600).toFixed(2);
        })
        .catch(err => {
          this.networkSpeed = " May have problems";
          this.overlay_name = false;

          let message = err.message;
          //   this.abstract = "Connection to PubMed is having a problem.";
          //   this.authorData.push(
          //     "It appears that the PubMed service is not available at this time. "
          //   );
          if (message === "Network Error") {
            // alert(
            //   "It appears that the PubMed service is not available at this time. "
            // );
            this.alert_error = true;
            setTimeout(this.removeErrorAlert, 3000);
          } else {
            // alert(message);
            this.alert_error = true;
            setTimeout(this.removeErrorAlert, 3000);
          }
          console.log(err);
        });
    }
  }
};
</script>

<style>
.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.title {
  font-family: "Quicksand", "Source Sans Pro", -apple-system, BlinkMacSystemFont,
    "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
  display: block;
  font-weight: 300;
  font-size: 100px;
  color: #35495e;
  letter-spacing: 1px;
}

.subtitle {
  font-weight: 300;
  font-size: 42px;
  color: #526488;
  word-spacing: 5px;
  padding-bottom: 15px;
}

.links {
  padding-top: 15px;
}
</style>