<template>
    <div>
    <div class="row text-center">
        <v-btn
            class="ma-2"
            color="primary"
            dark
            >
            Accept
            <v-icon
                dark
                right
                >
                mdi-checkbox-marked-circle
            </v-icon>
        </v-btn>
        <v-btn
            class="ma-2"
            color="red"
            dark
            >
            Decline
            <v-icon
                dark
                right
                >
                mdi-cancel
            </v-icon>
        </v-btn>
        <v-btn
            class="ma-2"
            dark
            @click="leaveRoom"
            >
            <v-icon
                dark
                left
                >
                mdi-minus-circle
            </v-icon>
            Leave Chat
        </v-btn>
    </div>
    <div class="row text-center">
        <div v-if="deactivateChat == false">
            <div class="col-md-6">
                <h3>Local</h3>
                <div id="my-video-chat-window" ref="myVideo" class="local-video"></div>
            </div>
            <div class="col-md-6">
                <h3>Remote</h3>
                <div id="remote-media-div" class="remote-video"></div>
            </div>
        </div>
        <div v-if="deactivateChat == true">
            <hr>
            <hr>
            <p class="text-center">This room was closed!</p>
            <hr>
            <hr>
        </div>
    </div>
</div>
</template>

<script>
import axios from 'axios'

export default {
    props: ['username', 'room'],
    name: 'VideoChat',
    data: function () {
        return {
            accessToken: '',
            videoChatWindow: '',
            activeRoom: '',
            deactivateChat: false,
        }
    },
    methods : {
        leaveRoom(){
            alert('iyg');
        },
        muteRoom : function(){
            this.activeRoom.disconnect();
            document.getElementById('my-video-chat-window').remove();
        },
        // leaveRoom : function(){
        //     this.activeRoom.disconnect();
        //     document.getElementById('remote-media-div').remove();
        //     document.getElementById('my-video-chat-window').remove();
        //     this.deactivateChat = true;
        //     console.log(this.activeRoom);
        // },
        getAccessToken : function () {
            const _this = this
            const axios = require('axios')

            // Request a new token
            axios.get(`http://localhost:3000/token?identity=${this.username}`)
                .then(function (response) {
                    _this.accessToken = response.data.token
                })
                .catch(function (error) {
                    console.log(error);
                })
                .then(function () {
                    _this.connectToRoom()
                });
        },
        connectToRoom : function () {

            const { connect, createLocalVideoTrack } = require('twilio-video');
            const _this = this;

            connect( this.accessToken, { name:this.room }).then(room => {

                console.log(`Successfully joined a Room: ${room}`);

                _this.activeRoom = room;

                const videoChatWindow = this.$refs["myVideo"];

                createLocalVideoTrack().then(track => {
                    videoChatWindow.appendChild(track.attach());
                });

                room.participants.forEach(participant => {
                 participant.on('trackSubscribed', track => {
                    document.getElementById('remote-media-div').appendChild(track.attach());
                  });
                });

                room.on('participantConnected', participant => {
                    console.log(`Participant "${participant.identity}" connected`);

                    participant.tracks.forEach(publication => {
                        console.log( publication.track, 'foreach');
                        if (publication.isSubscribed) {
                            const track = publication.track;
                            document.getElementById('remote-media-div').appendChild(track.attach());

                        }
                    });

                    participant.on('trackSubscribed', track => {
                        document.getElementById('remote-media-div').appendChild(track.attach());
                    });
                });
            }, error => {
                console.error(`Unable to connect to Room: ${error.message}`);
            });
        },
    },
    mounted : function () {
        console.log('Video chat room loading...')

        this.getAccessToken()
        this.videoChatWindow = document.getElementById('video-chat-window');
    }
}
</script>

<style media="screen">
    .col-md-6{
        width: 48%;
        float: left;
        margin-right: 20px;
    }
    .video-options{
        padding: 10px;
        margin: 15px;
        border: 1px solid #2980b9;
        border-radius: 5px;
        color: #2980b9;
    }
    .video-options a{
        font-size: 20px;
        margin-right: 20px;
    }
</style>
