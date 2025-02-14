<template>
  <v-app class="yellow lighten-2" dark >
    <v-content >    
    <v-parallax src="https://i.pinimg.com/originals/75/2a/30/752a309d15fac0b23d006d9b568aff0d.gif" width="100%" >
    <div class="fill-height bottom-gradient">
      <v-container grid-list-md text-xs-center mt-5>
        <v-layout class="justify-center" column align-center 
          row align-center
          text-xs-center
        >
          <v-flex fill-height mb-1 mt-5 >
            <h1 class="display-4 font-weight-black mb-3 yellow--text text--lighten-3">
              CROWDFUNDING
            </h1>
            <p class="title font-weight-black lime--text text--lighten-4">
              Utilizing Ethereum for Decentralized Crowdfunding
            </p>
          </v-flex>
        </v-layout>

        <v-layout row justify-center >
          <v-dialog v-model="startProjectDialog" max-width="600px" persistent>
            <v-btn round slot="activator" color="deep-purple accent-1" dark>Start a Project</v-btn>
            <v-card class="deep-purple accent-1 " dark>
              <v-card-title>
                <span class="headline font-weight-black mt-2 ml-4 orange--text text--lighten-3">Bring your project to life! ✨💫</span>
              </v-card-title>
              <v-card-text class="pt-0 font-weight-black">
                <v-container class="pt-0" grid-list-md>
                  <v-layout wrap>
                    <v-flex xs12>
                      <v-text-field
                        label="Title"
                        persistent-hint
                        v-model="newProject.title">
                      </v-text-field>
                    </v-flex>
                    <v-flex xs12>
                      <v-textarea
                        label="Description"
                        persistent-hint
                        v-model="newProject.description">
                      </v-textarea>
                    </v-flex>
                    <v-flex xs12 sm6>
                      <v-text-field
                        label="Amount Needed (ETH)"
                        type="number"
                        step="0.0001"
                        min="0"
                        v-model="newProject.amountGoal">
                      </v-text-field>
                    </v-flex>
                    <v-flex xs12 sm6>
                      <v-text-field
                        label="Duration (in days)"
                        type="number"
                        v-model="newProject.duration">
                      </v-text-field>
                    </v-flex>
                  </v-layout>
                </v-container>
              </v-card-text>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn
                  color="orange lighten-3"
                  flat
                  @click="startProjectDialog = false;
                  newProject.isLoading = false;">
                  Close
                </v-btn>
                <v-btn color="orange lighten-3"
                  flat
                  @click="startProject"
                  :loading="newProject.isLoading">
                  Save
                </v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-layout>
      </v-container>
      </div>
      </v-parallax>
      <v-container
        grid-list-lg
      >
        <h1 class="display-1 teal--text text--darken-3 font-weight-bold text-xs-center mb-3">
         👾 P R O J E C T S 👾
        </h1>
        <v-layout row wrap>
          <v-flex v-for="(project, index) in projectData" :key="index" xs12> 



            <v-dialog
              v-model="project.dialog"
              width="800"
            >
              <v-card >
                <v-card-title class="headline font-weight-black">
                  {{ project.projectTitle }}
                </v-card-title>
                <v-card-text >
                  {{ project.projectDesc }}
                </v-card-text>
                <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-btn
                    color="blue darken-1"
                    flat="flat"
                    @click="projectData[index].dialog = false"
                  >
                    Close
                  </v-btn>
                </v-card-actions>
              </v-card>
            </v-dialog>
            <v-hover>
              <v-card 
                slot-scope="{ hover }"
                :class="`elevation-${hover ? 10 : 2}`"
              >
                <v-card-title primary-title >
                  <div>
                    <div class="headline deep-purple--text text--accent-1 font-weight-black">
                      <v-chip
                        pill
                        :color="stateMap[project.currentState].color"
                        text-color="orange lighten-3" class="mt-0">
                      {{ stateMap[project.currentState].text }}
                      </v-chip>
                      {{ project.projectTitle }}
                    </div>
                    <br/>
                    <span>🌼 {{ project.projectDesc.substring(0, 100) }}</span>
                    <span v-if="project.projectDesc.length > 100">
                      ... <a @click="projectData[index].dialog = true">[Show full]</a>
                    </span>
                    <br/><br/>
                    <span>⏳ Up Until: <b>{{ new Date(project.deadline * 1000) }}</b></span>
                    <br/><br/>
                    <span>💰 Goal of <b>{{ project.goalAmount / 10**18 }} ETH </b></span>
                    <span v-if="project.currentState == 1">wasn't achieved before deadline</span>
                    <span v-if="project.currentState == 2">has been achieved</span>
                  </div>
                </v-card-title>
                <v-flex
                  v-if="project.currentState == 0 && account != project.projectStarter"
                  class="d-flex ml-3" xs12 sm6 md3>
                  <v-text-field
                    label="Amount (in ETH)"
                    type="number"
                    step="0.0001"
                    min="0"
                    v-model="project.fundAmount"
                  ></v-text-field>
                  <v-btn
                    class="mt-3"
                    color="orange"
                    @click="fundProject(index)"
                    :loading="project.isLoading"
                    outline
                  >
                    Fund
                  </v-btn>
                </v-flex>
                <v-flex class="d-flex ml-3" xs12 sm6 md3>
                  <v-btn
                    class="mt-3"
                    color="amber darken-1 white--text"
                    v-if="project.currentState == 1"
                    @click="getRefund(index)"
                    :loading="project.isLoading"
                  >
                    Get refund
                  </v-btn>
                </v-flex>
                <v-card-actions v-if="project.currentState == 0" class="text-xs-center">
                  <span class="font-weight-black" style="width: 200px;">
                    {{ project.currentAmount / 10**18 }} ETH
                  </span>
                  <v-progress-linear
                    height="10"
                    :color="stateMap[project.currentState].color"
                    :value="(project.currentAmount / project.goalAmount) * 100"
                  ></v-progress-linear>
                  <span class="font-weight-bold" style="width: 200px;">
                    {{ project.goalAmount / 10**18 }} ETH
                  </span>
                </v-card-actions>
              </v-card>
            </v-hover>
          </v-flex>
        </v-layout> 

      </v-container>

    </v-content>
  </v-app>
</template>

<script>
// We import our the scripts for the smart contract instantiation, and web3
import crowdfundInstance from '../contracts/crowdfundInstance';
import crowdfundProject from '../contracts/crowdfundProjectInstance';
import web3 from '../contracts/web3';

export default {
  name: 'App',
  data() {
    return {
      startProjectDialog: false,
      account: null,
      stateMap: [
        { color: 'primary', text: 'Ongoing' },
        { color: 'warning', text: 'Expired' },
        { color: 'success', text: 'Completed' },
      ],
      projectData: [],
      newProject: { isLoading: false },
    };
  },
  mounted() {
    // this code snippet takes the account (wallet) that is currently active
    web3.eth.getAccounts().then((accounts) => {
      [this.account] = accounts;
      this.getProjects();
    });
  },
  methods: {
    getProjects() {
      crowdfundInstance.methods.returnAllProjects().call().then((projects) => {
        projects.forEach((projectAddress) => {
          const projectInst = crowdfundProject(projectAddress);
          projectInst.methods.getDetails().call().then((projectData) => {
            const projectInfo = projectData;
            projectInfo.isLoading = false;
            projectInfo.contract = projectInst;
            this.projectData.push(projectInfo);
          });
        });
      });
    },
    startProject() {
      this.newProject.isLoading = true;
      crowdfundInstance.methods.startProject(
        this.newProject.title,
        this.newProject.description,
        this.newProject.duration,
        web3.utils.toWei(this.newProject.amountGoal, 'ether'),
      ).send({
        from: this.account,
      }).then((res) => {
        const projectInfo = res.events.ProjectStarted.returnValues;
        projectInfo.isLoading = false;
        projectInfo.currentAmount = 0;
        projectInfo.currentState = 0;
        projectInfo.contract = crowdfundProject(projectInfo.contractAddress);
        this.startProjectDialog = false;
        this.newProject = { isLoading: false };
      });
    },
    fundProject(index) {
      if (!this.projectData[index].fundAmount) {
        return;
      }

      const projectContract = this.projectData[index].contract;
      this.projectData[index].isLoading = true;
      projectContract.methods.contribute().send({
        from: this.account,
        value: web3.utils.toWei(this.projectData[index].fundAmount, 'ether'),
      }).then((res) => {
        const newTotal = parseInt(res.events.FundingReceived.returnValues.currentTotal, 10);
        const projectGoal = parseInt(this.projectData[index].goalAmount, 10);
        this.projectData[index].currentAmount = newTotal;
        this.projectData[index].isLoading = false;

        // Set project state to success
        if (newTotal >= projectGoal) {
          this.projectData[index].currentState = 2;
        }
      });
    },
    getRefund(index) {
      this.projectData[index].isLoading = true;
      this.projectData[index].contract.methods.getRefund().send({
        from: this.account,
      }).then(() => {
        this.projectData[index].isLoading = false;
      });
    },
  },
};
</script>
