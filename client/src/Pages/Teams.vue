<template>
    <div>
        <Navbar/>
        <div class="container" style="margin-top: 40px">
            <h3 class="float-left font-weight-bold text-gray-800 mb-1">Event {{this.$route.params.tag}}</h3>
            <router-link :to="{name: 'home'}" class="btn btn-sm btn-secondary text-white float-right">Back</router-link>
            <button data-toggle="modal" data-target="#eventModal" type="button" id="create-event" class="btn btn-sm btn-haaukins float-right mr-2">Restart</button>
            <div class="clearfix"></div>
            <hr>
            <div v-if="error" class="alert alert-danger alert-dismissible">{{error}}
                <button type="button" class="close" data-dismiss="alert" aria-label="Close">
                    <span aria-hidden="true">&times;</span>
                </button>
            </div>
            <div v-if="success" class="alert alert-success alert-dismissible">{{success}}
                <button type="button" class="close" data-dismiss="alert" aria-label="Close">
                    <span aria-hidden="true">&times;</span>
                </button>
            </div>
            <div v-if="loaderIsActive" class="alert myalert-loading alert-dismissible">
                <div class="d-inline mr-2">
                    <img class="loading-logo" src="../assets/bluelogo.png" width="50" height="50">
                </div>
                <div class="d-inline mr-2">{{loader_id}}</div>
                <div class="d-inline"><strong>{{loader_msg}}</strong></div>
                <button type="button" class="close" data-dismiss="alert" aria-label="Close" style="top: 13px;">
                    <span aria-hidden="true">&times;</span>
                </button>
            </div>
            <div class="table-responsive">
                <table class="table mx-auto table-hover table-striped" id="teamsEventTable" cellspacing="0" style="table-layout: auto; width: 100%">
                    <thead>
                        <tr class="text-center">
                            <td>#</td><td>Team_ID</td><td>Name</td><td>Email</td><td>Reset</td><td>Restart</td><td>Actions</td>
                        </tr>
                    </thead>
                    <tbody v-if="teams">
                        <tr v-for="(team,count) in teams.teamsList" v-bind:key="team.id">
                            <td class="text-center">{{count + 1}}</td>
                            <td><strong><router-link :to="{name: 'team', params: {id: team.id}}" class="text-haaukins" >{{team.id}}</router-link></strong></td>
                            <td>{{team.name}}</td>
                            <td>{{team.email}}</td>
                            <td class="text-center"><button class="btn btn-secondary btn-sm" v-on:click="resetFrontend(team.id)">Frontend</button></td>
                            <td class="text-center"><button class="btn btn-secondary btn-sm" v-on:click="restartTeamLab(team.id)">Lab</button></td>
                            <td class="text-center">
                                <button class="btn btn-warning btn-sm m-btn-responsive" v-on:click="suspendResumeTeamLab(team.id, true)">Suspend</button>
                                <button class="btn btn-warning btn-sm" v-on:click="suspendResumeTeamLab(team.id, false)">Resume</button>
                            </td>
                        </tr>
                    </tbody>
                    <tbody v-else>
                        <tr>
                            <td colspan="7" class="text-center">No Teams founds...</td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>
        <Footer/>
    </div>
</template>

<script>
    import Navbar from "../components/Navbar";
    import Footer from "../components/Footer";
    import { daemonclient } from "../App";
    import { ListEventTeamsRequest, RestartTeamLabRequest, ResetFrontendsRequest, Team, SetTeamSuspendRequest } from "daemon_pb"

    export default {
        name: "Teams",
        components: {Footer, Navbar},
        data: function () {
            return {
                error: null,
                success: null,
                teams: null,
                eventStatus: null,
                loaderIsActive: false,

                loader_msg:"Loading...",
                loader_id:""
            }
        },
        created() {
            this.listTeams();
        },
        methods: {
            listTeams () {
                let getRequest = new ListEventTeamsRequest();
                getRequest.setTag(this.$route.params.tag);
                daemonclient.listEventTeams(getRequest, {Token: localStorage.getItem("user")}, (err, response) => {
                    if (err == null) {
                        this.teams = response.toObject();
                    }else{
                        this.error = err['metadata']['grpc-message'];
                    }
                });
            },
            restartTeamLab (teamID) {
                const that = this
                this.loaderIsActive = true

                let getRequest = new RestartTeamLabRequest();
                getRequest.setEventtag(this.$route.params.tag);
                getRequest.setTeamid(teamID)

                const call = daemonclient.restartTeamLab(getRequest, {Token: localStorage.getItem("user")});

                call.on('data', function(response) {
                    //this.status = response.getErrormessage();
                    window.console.log(response)
                });
                call.on('end', function() {
                    window.console.log("enddd")
                });
                call.on('error', function(e) {
                    that.error = e
                });
                call.on('status', function(status) {
                    that.loaderIsActive = false
                    if (status['metadata']['grpc-message'] == "") {
                        that.success = "Team Lab Successfully Restarted!"
                        that.listEvent()
                    }else{
                        that.error = status['metadata']['grpc-message']
                    }
                });
            },
            resetFrontend(teamID){
                const that = this
                this.loaderIsActive = true

                let getRequest = new ResetFrontendsRequest();
                let getTeam = new Team();

                getTeam.setId(teamID);
                getRequest.setEventtag(this.$route.params.tag);
                getRequest.addTeams(getTeam);

                const call = daemonclient.resetFrontends(getRequest, {Token: localStorage.getItem("user")});

                call.on('data', function(response) {
                    //this.status = response.getErrormessage();
                    window.console.log(response.getStatus())

                    that.loader_msg = response.getStatus()
                    that.loader_id = response.getTeamid()

                });
                call.on('error', function(e) {
                    that.error = e
                });
                call.on('status', function() {
                    setTimeout(function(){
                        that.loaderIsActive = false
                        that.success = "Frontend successfully restarted!"
                    }, 1000);
                });
            },
            suspendResumeTeamLab: function (teamID, setSuspend) {
                let getRequest = new SetTeamSuspendRequest()
                getRequest.setTeamid(teamID)
                getRequest.setEventtag(this.$route.params.tag)
                getRequest.setSuspend(setSuspend)

                daemonclient.setTeamSuspend(getRequest, {Token: localStorage.getItem("user")}, (err) => {
                    if (err == null) {
                        this.success = "Action completed!"
                    }else{
                        this.error = err["message"];
                    }
                });
            }
        }
    }
</script>

<style scoped>
    .btn-nobox {
        box-shadow: none !important;
        outline: 0;
    }
    .list-group-item span {
        border: solid #222;
        border-width: 0 1px 1px 0;
        display: inline;
        cursor: pointer;
        padding: 5px;
        position: absolute;
        right: 0;
        margin-top: 10px;
    }

    .list-group-item a.btn.collapsed span {
        transform: rotate(40deg);
        -webkit-transform: rotate(40deg);
        transition: .3s transform ease-in-out;
    }

    .list-group-item a.btn span {
        transform: rotate(-140deg);
        -webkit-transform: rotate(-140deg);
        transition: .3s transform ease-in-out;
    }

    .m-btn-responsive{
        margin-right: 5px;
    }

    @media (max-width: 992px) {
        .m-btn-responsive{
            margin-right: 0px;
            margin-bottom: 5px;
        }
    }

</style>