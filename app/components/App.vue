<template>
    <Page>
        <ActionBar title="Expense Tracker"/>
        <GridLayout colums="*" rows="*">
            <TextField :text="expense.description" hint="Expense Description" maxLength="25" />
            <TextField :text="expense.amount" hint="Amount"  />
            <Button text="Add Expense" @tap="addExpense" />
            <ListView for="expense in expenses" @itemTap="deleteExpense($index)">
              <v-template>
                <Label :text="expense.amount" />
                <Label :text="expense.description" />
              </v-template>
            </ListView>
        </GridLayout>
    </Page>
</template>

<script>
  export default {
    created(){
      var Sqlite  = require( "nativescript-sqlite" );
      var that = this;
      this.sql = new Sqlite("expenses",(err,db) => {
        if(err){
          console.log("Oops something went wrong!")
        }
      }).then((db) => {
        that.db = db;
        that.createTable();
      });
    },
    data() {
      return {
        msg: 'Hello World!',
        expenses: [],
        expense: {
          amount: 0.00,
          description: '',
        },
        sql: {},
        db: {}
      }
    },
    methods:{
      createTable: function(){
        that = this;
        this.db.version((err,ver) => {
          if (ver == 0){
            that.db.execSQL('create table expenses(
              id integer PRIMARY KEY,
              amount text NOT NULL,
              description text NOT NULL
            )');
          }
          else{
            that.getExpenses();
          }
        })
      },
      addExpense: function(){
        //SQL insert query
        var that = this;
        this.db.execSQL("INSERT into expenses (amount,description) values (?)",[this.expense.amount,this.expense.description],(err,row) =>{
          if (err){
            console.log("Ooops something went wrong")
          }
          else{
            that.expenses.push(row);
          }
        })
        //push to expenses array
      },
      deleteExpense: function(index){
        //SQL delete query
        var that = this;
        this.db.execSQL("delete from expenses where (id)=(?)",[this.expenses[index].id],(err,row) => {
          if (err){
            console.log("Oops something went wrong")
          }
          else{
            that.expenses.splice(index,1);
          }
        })
        //remove from expenses array
      },
      getExpenses: function(){
        //SQL get query
        var that = this;
        this.db.get('SELECT * from expenses', (err,row) =>{
            //push to expense array
            row.forEach((item) =>{
              that.expenses.push(item);
            })
        })

      }
    }
  }
</script>

<style scoped>
    ActionBar {
        background-color: #53ba82;
        color: #ffffff;
    }

    .message {
        vertical-align: center;
        text-align: center;
        font-size: 20;
        color: #333333;
    }
</style>
