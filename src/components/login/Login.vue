<template>
  <a-row v-if="$system_variables.status_task_loaded == 1" v-show="$system_variables.status_data_loaded == 1">
      <a-col :lg="{ span: 11, offset: 7 }" :sm="{ span: 13, offset: 5 }"> 
        <a-alert v-if="alert_message != ''"            
            :type="alert_type"
            closable
            >
            <p slot="description" v-html="alert_message">
            </p>
        </a-alert>             
        <a-form-model ref="formLogin" id="formLogin" :model="loginInputs" :rules="loginInputRules" :label-col="{ span: 8 }" :wrapper-col="{ span: 14 }">
            <a-form-model-item has-feedback :label="$system_functions.get_label_task('label_email')" prop="email">
              <a-input name='email' v-model="loginInputs.email" autocomplete="off" />
            </a-form-model-item>
            <a-form-model-item has-feedback :label="$system_functions.get_label_task('label_password')">
                <a-input-password name='password' v-model="loginInputs.password" type="password" autocomplete="off" />
            </a-form-model-item>
            <a-form-model-item :wrapper-col="{offset: 18 }">
                <a-button type="primary" @click="login">
                    {{$system_functions.get_label('label_login')}}
                </a-button>                
            </a-form-model-item>
          </a-form-model>
      </a-col>
    </a-row>
</template>

<script>

export default {
    name: 'Login',
    components: { },
    mounted:function(){
        this.$system_functions.load_task_languages([
            {language:this.$system_variables.language,file:'components/login/language.js'},
        ]);
        this.init();
    },
    data() {
        return {                        
            alert_message: '',
            alert_type:'error',
            otp_required: false,
            token_sms: '',
            loginInputs:{'email':'','password':''},
            loginInputRules: {
                email: [{ required: true, message: this.$system_functions.get_label_task('msg_email_require'), trigger: 'blur' }],
                        
                        }

            // form_title: 'Fill out the form below to login.',
            // otp_error_message: ''
        }
    },
    methods:{    
        init: function()
        {   
            this.$system_variables.status_task_loaded=1;
            this.$system_variables.status_data_loaded=1;
        },               
        login: function(event)
        {
            this.$refs.formLogin.validate(valid => {
            if (valid) 
            {
                this.alert_message = '';
                this.$system_variables.status_data_loaded = 0;
                var formData=new FormData(document.getElementById('formLogin'));
                formData.append ('language', this.$system_variables.language);                
                //this.$axios.post('/user/login',formData)
                this.$axios.post('user/login',formData)
                    .then(response=>{                        
                        this.$system_variables.status_data_loaded = 1;
                        if(response.data.errorStr == '')
                        {
                            localStorage.setItem('api_token', response.data.user.api_token);                            
                            this.$system_functions.set_user(response.data.user);
                            this.$system_functions.setAxiosHeader();
                            this.$router.push("/");
                        }
                        else
                        {
                            this.alert_message = response.data.errorStr;
                            this.alert_type = 'error';
                        }
                        this.$system_variables.status_data_loaded = 1;
                    })
                    .catch(error => {
                        //console.log(error.response);
                        this.alert_type = 'error';
                        
                        if(error.response && error.response.data && error.response.data.errorStr)
                        {
                            if(error.response.data.errorStr == 'LOGIN_FAILED')
                            {
                                this.alert_message = this.$system_functions.get_label('USER_NOT_FOUND');
                            }
                            else
                            {
                                this.$system_functions.responseErrorTask(error.response.data.errorStr);                                
                            }
                        }
                        else
                        {
                            this.$system_functions.responseErrorTask();//default Error
                        }
                        
                        this.$system_variables.status_data_loaded = 1;
                    });                
                } 
                
            });
            
        },
        
    }
}
</script>