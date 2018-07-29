<template>
   <textarea ref="txt" class="txt">
   select * from ParamsAtos
   </textarea>
</template>

<script>
import CodeMirror from "codemirror";
import "codemirror/lib/codemirror.css";
import "codemirror/theme/neo.css";
import "codemirror/theme/panda-syntax.css";
import "codemirror/mode/sql/sql.js";

import "codemirror/addon/hint/show-hint.js";
import "codemirror/addon/hint/show-hint.css";
import "./sql-hint.css";
//import "codemirror/addon/hint/sql-hint.js";

export default {
  name: "SqlEditor",
  data() {
    return {
      editor: null,
      //autocompletion: the dictionary of all possible keywords
      //[list complete of reserved keyword for MSSQL](https://docs.microsoft.com/en-us/sql/t-sql/language-elements/reserved-keywords-transact-sql?view=sql-server-2017)
      dico: [
        { className: "sql", text: "SELECT" },
        { className: "sql", text: "FROM" },
        { className: "sql", text: "WHERE" },
        { className: "sql", text: "INNER" },
        { className: "sql", text: "JOIN" },
        { className: "sql", text: "UNION" },
        { className: "sql", text: "EXEC" },
        { className: "sql", text: "INSERT" },
        { className: "sql", text: "INTO" },
        { className: "sql", text: "VALUES" },
        { className: "sql", text: "UPDATE" },
        { className: "sql", text: "DELETE" },
        { className: "sql", text: "GROUP" },
        { className: "sql", text: "BY" },
        { className: "sql", text: "HAVING" },
        { className: "sql", text: "IS" },
        { className: "sql", text: "DISTINCT" },
        { className: "sql", text: "OUTER" },
        { className: "sql", text: "TOP" },
        { className: "sql", text: "EXISTS" },
        { className: "sql", text: "WHEN" },
        { className: "sql", text: "CASE" },
        { className: "sql", text: "CAST" },
        { className: "sql", text: "IN" },
        { className: "sql", text: "NULL" },
        { className: "table", text: "te_cash_exchange_new" },
        { className: "table", text: "ParamsAtos" },
        { className: "table", text: "te_client_transfers" },
        { className: "column", text: "status_cash" },
        { className: "column", text: "datet" },
        { className: "column", text: "ammount" },
        { className: "column", text: "cash_exchange_id_start" },
        { className: "column", text: "cash_exchange_id_end" },
        { className: "pf", text: "AddParamAtos" },
        { className: "pf", text: "verify_backoffice_user" },
        { className: "pf", text: "checkAllowOperation" },
        { className: "pf", text: "addMoneyIn_01" }
      ]
    };
  },
  methods: {
    /*
    Return a list of suggestion base on the lineContext and a searchString. 
    Input:
    - searchString is the current word that user is typing
    - lineContext is the content of line from first character to the current word (searchString)
    Output: a list of suggestion: each suggestion is an object {text, displayText, className}. See https://codemirror.net/doc/manual.html#addon_show-hint
    - keywords start with the searchString appears first in the suggestion list
    - lineContext (matching pattern) to narrow down the list
    */
    suggest(lineContext, searchString) {
      searchString = searchString.toLowerCase();
      let ignoreTable = false;
      if (lineContext) {
        //analyze the lineContext (TODO)
        lineContext = lineContext.toLowerCase();
        if (lineContext == "select ") {
          ignoreTable = true;
        }
      }

      let startsWithList = [];
      let containsList = [];
      this.dico.forEach(suggestion => {
        //eliminate suggestion base on searchString context analysis
        if (ignoreTable && suggestion.className == "table") return;
        let keyword = suggestion.text.toLowerCase();
        if (keyword.startsWith(searchString)) startsWithList.push(suggestion);
        else if (keyword.includes(searchString)) containsList.push(suggestion);
      });
      startsWithList.push(...containsList);
      return startsWithList;
    },
    /*
    [hint implementation for codemirror](https://codemirror.net/doc/manual.html#addon_show-hint):
    take an editor instance and options object, and return a {list, from, to} object, where list is an array of strings or objects (the completions), and from and to give the start and end of the token that is being completed as {line, ch} objects. 
     */
    hint(editor, _) {
      let cur = editor.getCursor();
      let token = editor.getTokenAt(cur);
      let searchString = token.string;
      let line = editor.getLine(cur.line);
      let searchContext = line ? line.substring(0, token.start) : null;
      return {
        list: this.suggest(searchContext, searchString),
        from: CodeMirror.Pos(cur.line, token.start),
        to: CodeMirror.Pos(cur.line, token.end)
      };
    }
  },
  mounted() {
    this.editor = CodeMirror.fromTextArea(this.$refs.txt, {
      tabSize: 4,
      mode: "text/x-mysql",
      theme: "panda-syntax",
      lineNumbers: true,
      line: true,
      lineWrapping: true,
      hintOptions: {
        completeSingle: false,
        hint: this.hint
      },
      extraKeys: {
        "Ctrl-Space": editor => {
          editor.showHint();
        }
      }
    });
    this.editor.on("keypress", (editor, _) => {
      editor.showHint();
    });
  }
};
</script>

<style scoped>
.txt {
  width: 100%;
  height: 100%;
}
</style>
