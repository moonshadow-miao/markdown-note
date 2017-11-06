###
bug1:在一个组件中通过bus总线监听事件,然后组件每加载一次后,bus监听的事件会多触发一次,同一请求触发两次或者更多
解决:在组件中监听事件的时候,在离开当前组件的时候,应该要解绑事件.	
 	
	mounted(){
      bus.$on('changeAccount', ()=> {
        var id = this.id;
        Account_detail({resourceAccId: id}).then(res=> {
          this.accountInfo = res;
        });
        Account_appResource({resourceAccId: id}).then(res=> {
          this.appList = res
        });
        Account_getDistribution({resourceAccId: id}).then(res=>{
          this.accountList =res.data;
        })
      });
    },
	destroyed(){
      bus.$off('changeAccount');
    },

