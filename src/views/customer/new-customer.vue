<style>
  .vw-widget-body {
    text-align: -webkit-center;
  }
  .form {
    width: 70%;
  }

  #state{
    width: 100%;
    background: white;
    height: 32px;
    border-color: #e2e2e2;
    border-radius: 4px;
   }
   #country{
    width: 100%;
    background: white;
    height: 32px;
    border-color: #e2e2e2;
    border-radius: 4px;
   }
</style>

<template>

    <!--<Widget>
      <WidgetHeading :id="1" :Title="'Add New Customer'" :TextColor="true" :DeleteButton="false" :ColorBox="true" :Expand="true" :Collapse="true" :Editable='false'></WidgetHeading>
      <WidgetBody>-->
  <Row>
    <Col span="12" offset="6">
      <Card style="padding:10px;">
        <p slot="title">Add New Customer</p>
          <Form class="form" label-position="left" ref="formValidate" :model="formValidate" :rules="ruleValidate" :label-width="140" style="width:100%">
            <FormItem label="Configuration Name" prop="configuration">
              <Select v-model="formValidate.configuration" style="width:100%;text-align:left" @on-change="configChange">
                <Option  v-for="item in configs" :value="item.id" :key="item">{{ item.configName }} ({{item.domain}})</Option>
              </Select>
            </FormItem>
            <FormItem label="Name" prop="name" style="display:none;" id="CustomerName">
                <Input v-model="formValidate.name" placeholder="Enter your name"></Input>
            </FormItem>
            <FormItem label="E-mail" prop="mail">
                <Input v-model="formValidate.mail" placeholder="Enter your e-mail"></Input>
            </FormItem>
            <FormItem label="Address" prop="AddressLine1">
              <Row>
                <Col span="12">
                  <FormItem prop="AddressLine1">
                      <Input v-model="formValidate.AddressLine1" placeholder="AddressLine1"></Input>
                  </FormItem>
                </Col>
                <Col span="12">
                  <FormItem>
                    <Input v-model="formValidate.AddressLine2" placeholder="AddressLine2"></Input>
                  </FormItem>
                </Col>
              </Row>
            </FormItem>
            <FormItem label="Country" prop="country">
                <select v-model="formValidate.country" id="country" name ="country">
                </select>
            </FormItem>
            <FormItem label="State" prop="state" class="state1" style="display:none">
              <select v-model="formValidate.state" name ="state" id ="state" ></select>
            </FormItem>
            <FormItem label="City" prop="city">
              <Input v-model="formValidate.city" placeholder="Enter your city"></Input>
            </FormItem>
            <FormItem label="Zip Code" prop="PostalCode">
              <Input v-model="formValidate.PostalCode" placeholder="Enter Zip Code"></Input>
            </FormItem>
            <FormItem label="Mobile" prop="mobile">
              <Input v-model="formValidate.mobile" placeholder="Enter your Mobile No"></Input>
            </FormItem>
            <!-- <FormItem label="Phone" prop="phone">
              <Input v-model="formValidate.phone" placeholder="Enter your Phone No"></Input>
            </FormItem>
            <FormItem label="Fax" prop="fax">
                <Input v-model="formValidate.fax" placeholder="Enter your fax"></Input>
            </FormItem> -->
            <div style="text-align:center;">
              <Button type="primary" @click="handleSubmit('formValidate')" :loading="loading">Submit</Button>
              <Button type="ghost" @click="handleReset('formValidate')" style="margin-left: 8px;">Reset</Button>
            </div>
            </FormItem>
          </Form>
          <!--</WidgetBody>
          </Widget>-->
      </Card>
    </Col>
  </Row>

</template>

<script>
  import config from '@/config/customConfig.js'
  import axios from 'axios'
  import Cookies from 'js-cookie';
  import Vue from 'vue'
  import VueWidgets from 'vue-widgets'
  import 'vue-widgets/dist/styles/vue-widgets.css'
  import _ from 'lodash'
  import psl from 'psl'
  let settingId;
  Vue.use(VueWidgets);
  export default {
    name: 'new-customer',
    data () {
      const validateNum = async(rule, value, callback) => {
        let patt = new RegExp('^[0-9]+$')
        let _res = patt.test(value)
        if (!_res) {
          callback(new Error('Not Allowed Special Character or string'))
        } else {
          callback();
        }
      };
      // const validateBlank = async(rule, value, callback) => {
      //   let blankPatt = /^\S*$/g
      //   // console.log("------------value", typeof value)
      //   let blankTest = value.match(blankPatt)
      //   // console.log("======================_res",blankTest);
      //   if (!blankTest) {
      //     callback(new Error('Password Does not Allow Spaces'))
      //   } else {
      //     callback();
      //   }
      // };
      const validateBlank = (rule, value, callback) => {
        if (value.trim() === '') {
          callback(new Error('Space is not allowed'));
        } else {
          callback();
        }
      };
      return {
          loading: false,
          customCustomerUrl:'',
          customInvoiceUrl: '',
          formValidate: {
              configuration: '',
              name: '',
              mail: '',
              mobile : '',
              AddressLine1: '',
              AddressLine2 : '',
              city: '',
              state: '',
              country: '',
              PostalCode: ''
          },
          configs:[],
          ruleValidate: {
              name: [
                { required: true, message: 'The name cannot be empty', validator: validateBlank, trigger: 'blur' }
              ],
              configuration: [
                { required: true, message: 'Please choose configuration name', trigger: 'chahnge' }
              ],
              AddressLine1:[

                  { required: true, message: 'The addressline1 cannot be empty', trigger: 'blur' }
              ],

              city:[
                { required: true, message: 'The city cannot be empty', validator: validateBlank, trigger: 'blur' }
              ],
              state: [
                { required: true, message: 'Please select state', trigger: 'blur' }
              ],
              country:[
                { required: true, message: 'Please select Country', trigger: 'blur' }
              ],
              PostalCode:[
                { required: true, message: 'The Zip Code cannot be empty', validator: validateBlank, trigger: 'blur' },
                { validator: validateNum, trigger: 'blur' },
                {
                    max: 6 ,min:6, message: 'Please Enter 6 digit Zip Code', trigger: 'blur' 
                }
              ],
              mail: [
                  { required: true, message: 'Mailbox cannot be empty', trigger: 'blur' },
                  { type: 'email', message: 'Incorrect email format', trigger: 'blur' }
              ],
              mobile: [
                  { required: true, message: 'Mobile number can not be empty', validator: validateBlank, trigger: 'blur' },
                  { validator: validateNum, trigger: 'blur' },
                  { max: 10 ,min:10, message: 'Please Enter 10 digit mobile number', trigger: 'blur' }
              ]
              // phone: [
              //     { required: true, message: 'Phone no cannot be empty', trigger: 'blur' },
              //     { validator: validateNum, trigger: 'blur' }
              // ],
              // fax: [
              //     { required: true, message: 'Mobile no cannot be empty', trigger: 'blur' },
              //     { validator: validateNum, trigger: 'blur' }
              // ]
          }
      }
    },
    methods: {
        handleSubmit (name) {
            // // console.log('this.$refs[name]',this.$refs[name].validate())
            this.$refs[name].validate((valid) => {
                if (valid) {
                    //this.$Message.success('Success!');
                    this.createCustomer();
                } else {
                    
                    this.$Message.error('Please fill up all the fields correctly');
                }
            })
        },
        handleReset (name) {
            this.$refs[name].resetFields();
        },
        async settingData () {
          let self = this
          await axios.get(config.default.serviceUrl + 'settings?isActive=true', {
            headers:{
              Authorization : Cookies.get('auth_token'),
              subscriptionId : Cookies.get('subscriptionId')
            }
          })
          .then(function (response) {
            // console.log("response >>>>>>>>>>>>>>>>",response)
            //resp = response.data
            
            if (response.data.data.length != 0)
            {
              var newConf = response.data.data
              // console.log("self.configs---------------->before",newConf)
              self.configs = _.sortBy(newConf, ['configName']);
              // console.log("self.configs---------------->after",self.configs)
            }else
            {
                self.$Modal.warning({
                title: 'No Configuration available',
                okText : "Go to Settings",
                content: '<h3 style="font-family: initial;">Please navigate to settings and configure or activate at least one Xero or Quickbook account </h3>',
                onOk: () => {
                      self.$router.push({
                          name: 'Settings'
                      })
                  }
                });
            }
          })
          .catch(function (error) {
            // console.log("error",error);
            if(error.message == 'Network Error'){
                self.$Notice.error({
                    title: "Error",
                    desc: 'API service unavailable',
                    duration: 10
                })
            }else if(error.response.status == 403){
              self.$Notice.error(
                  {duration:0, 
                  title: error.response.statusText,
                  desc:error.response.data.message+'. Please <a href="'+config.default.flowzDashboardUrl+'/subscription-list" target="_blank">Subscribe</a>'}
                  );
              }else if(error.response.status == 401){
                
                let location = psl.parse(window.location.hostname)
                location = location.domain === null ? location.input : location.domain
                
                Cookies.remove('auth_token' ,{domain: location}) 
                self.$store.commit('logout', self);
                
                self.$router.push({
                    name: 'login'
                });
                self.$Notice.error({
                        title: error.response.data.name,
                        desc: error.response.data.message,
                        duration: 10
                    })
            }else {
              self.$Notice.error({
                title: error.response.data.name,
                desc: error.response.data.message,
                duration: 10
              })
            }
          });
          
        },
        configChange(data){
          $('#CustomerName').css("display","block")
          settingId = data;
          
        },
        async createCustomer () {
          this.loading = true;
          var self = this;
          let ConfigName = _.filter(this.configs , function(o){
              return o.id == settingId
          });
          let configName = ConfigName[0].configName;
          // console.log(">>>>>>>>>>> " , settingId)
          await axios({
              method:'get',
              url: config.default.serviceUrl + 'settings/'+settingId,
              headers:{
                Authorization : Cookies.get('auth_token'),
                subscriptionId : Cookies.get('subscriptionId')
              },
            })
            .then(async function(response) {
                // console.log(response)
                if(response.data.domain == 'custom'){
                    var params = {
                      settingId : settingId,
                      Name: self.formValidate.name.trim(),
                      ContactStatus: 'ACTIVE',
                      EmailAddress:self.formValidate.mail.trim(),
                      Address: self.formValidate.AddressLine1.trim()+","+self.formValidate.AddressLine2.trim()+","+self.formValidate.city.trim()+","+self.formValidate.state+","+self.formValidate.PostalCode.trim()+","+self.formValidate.country,
                      PhoneNumber:self.formValidate.mobile.trim()
                    }
                    // console.log("contact post params",params);

                      self.customCustomerUrl = response.data.customer_url;
                      self.customInvoiceUrl = response.data.invoice_url;
                      
                      axios({
                        method: 'post',
                        url: self.customCustomerUrl,
                        data: params,
                        headers:{
                          Authorization : Cookies.get('auth_token')
                        }
                      })
                      .then(function (response) {
                        self.$Message.success('created customer successfully');
                        self.$refs['formValidate'].resetFields();
                        self.loading = false;
                        self.$router.push({
                          name:'Customer List'
                        })
                      })
                      .catch(function (error) {
                        self.loading = false;
                        // console.log("error",error);
                        if(error.hasOwnProperty('response') && error.response.hasOwnProperty('status') && error.response.status == 401){
                            let location = psl.parse(window.location.hostname)
                            location = location.domain === null ? location.input : location.domain
                            
                            Cookies.remove('auth_token' ,{domain: location}) 
                            Cookies.remove('subscriptionId' ,{domain: location}) 
                            self.$store.commit('logout', self);
                            
                            self.$router.push({
                                name: 'login'
                            });
                            self.$Notice.error({
                                        title: error.response.data.name,
                                        desc: error.response.data.message,
                                        duration: 10
                                    })
                        }else if(error.hasOwnProperty('response') && error.response.hasOwnProperty('status') && error.response.status == 403){
                            self.$Notice.error({
                                title: error.response.statusText,
                                desc: error.response.data.message+'. Please <a href="'+config.default.flowzDashboardUrl+'/subscription-list" target="_blank">Subscribe</a>',
                                duration: 4.5
                            })
                        }else {
                            self.$Notice.error({
                                title: error.response.data.name,
                                desc: error.response.data.message,
                                duration: 10
                            })
                        }
                      });

                }else{
                    var params1 = {
                      settingId : settingId,
                      Name: self.formValidate.name.trim(),
                      ContactStatus: 'ACTIVE',
                      EmailAddress:self.formValidate.mail.trim(),
                      AddressLine1:self.formValidate.AddressLine1.trim(),
                      AddressLine2:self.formValidate.AddressLine2.trim(),
                      City:self.formValidate.city.trim(),
                      Country:self.formValidate.country,
                      PostalCode:self.formValidate.PostalCode.trim(),
                      PhoneNumber:self.formValidate.mobile.trim()
                    }
                      axios({
                        method: 'post',
                        url: config.default.serviceUrl + 'contacts',
                        data: params1,
                        headers:{
                          Authorization : Cookies.get('auth_token')
                        }
                      })
                      .then(function (response) {
                        // console.log('contact post response------------------',response)
                        self.loading = false;
                        self.$Message.success('created customer successfully');
                        self.$router.push({
                          name:'Customer List'
                        })
                        self.$refs['formValidate'].resetFields();
                      })
                      .catch(function (error) {
                        // console.log("error",error.response);
                        self.loading = false;
                        if(error.hasOwnProperty('response') && error.response.hasOwnProperty('status') && error.response.status == 401){
                            let location = psl.parse(window.location.hostname)
                            location = location.domain === null ? location.input : location.domain
                            
                            Cookies.remove('auth_token' ,{domain: location}) 
                            Cookies.remove('subscriptionId' ,{domain: location}) 
                            self.$store.commit('logout', self);
                            
                            self.$router.push({
                                name: 'login'
                            });
                            self.$Notice.error({
                                        title: error.response.data.name,
                                        desc: error.response.data.message,
                                        duration: 10
                                    })
                        }else if(error.hasOwnProperty('response') && error.response.hasOwnProperty('status') && error.response.status == 403){
                            self.$Notice.error({
                                title: error.response.statusText,
                                desc: error.response.data.message+'. Please <a href="'+config.default.flowzDashboardUrl+'/subscription-list" target="_blank">Subscribe</a>',
                                duration: 4.5
                            })
                        }else if (error.response.data.name === 'GeneralError') {
                          self.$Notice.error({
                                title: 'Credential Error',
                                desc: 'Account credential incorrect for <b>'+configName+'</b>',
                                duration: 10
                            })
                        }else {
                            self.$Notice.error({
                                title: error.response.data.name,
                                desc: error.response.data.message,
                                duration: 10
                            })
                        }
                        // self.$Message.error('error in create customer')
                      });
                }
            })
            .catch(function (error) {
              // console.log("error",error);
              self.loading = false;
              if(error.response.status == 403){
                self.$Notice.error({
                  duration:0, 
                  title: error.response.statusText,
                  desc:error.response.data.message+'. Please <a href="'+config.default.flowzDashboardUrl+'/subscription-list" target="_blank">Subscribe</a>'
                });
              }else if(error.response.status == 401){
                let location = psl.parse(window.location.hostname)
                location = location.domain === null ? location.input : location.domain
                
                Cookies.remove('auth_token' ,{domain: location}) 
                self.$store.commit('logout', self);
                
                self.$router.push({
                    name: 'login'
                });
                self.$Notice.error({
                  title: error.response.data.name,
                  desc: error.response.data.message,
                  duration: 10
                })
              }else {
                self.$Notice.error({
                  title: error.response.data.name,
                  desc: error.response.data.message,
                  duration: 10
                })
              }
            });     
          
        }
    },
    mounted(){
        this.settingData ();
        populateCountries("country", "state");
        $("#country").on("change",function() {
        // $('#country').change(function(){ 
            // var value = $(this).val();
            $('.state1').css("display","block")
        });
    }
  }
</script>
