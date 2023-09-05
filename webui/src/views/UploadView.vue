<script>
import LogModal from "../components/Logmodal.vue";
import SuccessMsg from "../components/SuccessMsg.vue"

export default {
    components: { LogModal, SuccessMsg },
    data: function () {
        return {
            errormsg: null,
            successmsg: null,
			detailedmsg: null,
            username: localStorage.getItem('username'),
            token: localStorage.getItem('token'),
            newUsername: "",
            profile: {
                requestId: 0,
                id: 0,
                username: "",
                followersCount: 0,
                followingCount: 0,
                photoCount: 0,
            },
            photoList: {
                requestUser: 0,
                identifier: 0,
                photos: [
                    {
                        id: 0,
                        userId: 0,
                        file: "",
                        date: "",
                        likesCount: 0,
                        commentsCount: 0,
                        likeStatus: null,
                        comment: "",
                    }
                ],
            },
            user: {
                id: 0,
                username: "",
            },
            comment: "",
            photoComments: {
                requestIdentifier: 0,
                photoIdentifier: 0,
                identifier: 0,
                comments: [
                    {
                        id: 0,
                        userId: 0,
                        photoId: 0,
                        photoOwner: 0,
                        ownerUsername: "",
                        username: "",
                        content: "",
                    }
                ],
            },
        }
    },
    methods: {

        async refresh() {
            await this.userProfile();
            await this.userPhotos();
        },
        async uploadFile() {
			this.images = this.$refs.file.files[0]
			/*this.submitFile();*/
		},

        async submitFile() {
			if (this.images === null) {
				this.errormsg = "Please select a file to upload."
			} else {
				try {
					let response = await this.$axios.put("/users/" + this.username + "/photo/" + Math.floor(Math.random() * 10000), this.images, {
						headers: {
							Authorization: "Bearer " + localStorage.getItem("token")
						}
					})
                    
					this.profile = response.data
					this.successmsg = "Photo uploaded successfully."
                    this.refresh();
				} catch (e) {
					if (e.response && e.response.status === 400) {
						this.errormsg = "Form error, please check all fields and try again. If you think that this is an error, write an e-mail to us.";
						this.detailedmsg = null;
					} else if (e.response && e.response.status === 500) {
						this.errormsg = "An internal error occurred. We will be notified. Please try again later.";
						this.detailedmsg = e.toString();
					} else {
						this.errormsg = e.toString();
						this.detailedmsg = null;
					}
				}
			}
		},
        async ViewProfile() {
			this.$router.push({ path: '/users/' + this.username + '/profile' })
		},

        async doLogout() {
			localStorage.removeItem("token")
			localStorage.removeItem("username")
			this.$router.push({ path: '/' })
		}

    }
}

</script>


<template>
    <nav id="sidebarMenu" class="col-md-2 d-md-block bg-light sidebar collapse">
		<div class="position-sticky pt-3 sidebar-sticky">
    		<h6 class="sidebar-heading d-flex justify-content-center align-items-center px-2 mt-3 mb-1 text-muted text-uppercase" style="font-size: 25px;" >
        		<span style="text-decoration: underline;">General</span>
			</h6>
            <ul class="nav flex-column">
                <li class="nav-item">
                    <RouterLink to="/session" class="nav-link d-flex align-items-center">
                        <svg class="feather">
                            <use href="/feather-sprite-v4.29.0.svg#home" />
                        </svg>
                        <span style="font-size: 20px;">Home</span>
                    </RouterLink>
                </li>
                <li class="nav-item">
                <Router-link :to="`/users/${username}/profile`" class="nav-link d-flex align-items-center" @click="ViewProfile">
                    <svg class="feather">
                        <use href="/feather-sprite-v4.29.0.svg#user" />
                    </svg>
                    <span style="font-size: 14px;">Profile</span>
                </Router-link>
            </li>
                <li class="nav-item">
                    <Router-link to="/" class="nav-link d-flex align-items-center" @click="doLogout">
                    <div style="border: 2px solid red; padding: 5px; border-radius: 5px;">
                        <span style="font-size: 20px; color: red;">Logout</span>
                    </div>
                    </Router-link>
                </li>
            </ul>
        </div>
    </nav>


    <div class="d-flex justify-content-between flex-wrap flex-md-nowrap align-items-center pt-3 pb-2 mb-3 border-bottom">
			<h1 class="h2" style="margin-left: 590px;">Welcome to the <span style="color: #8A3FFC;">Upload Section</span></h1>
			<div class="btn-group">
				<button class="btn btn-primary dropdown-toggle" type="button" id="dropdownMenuButton" data-bs-toggle="dropdown" aria-haspopup="true" aria-expanded="false">
					Menu
				</button>
		
					<div class="dropdown-menu" aria-labelledby="dropdownMenuButton">
					    <button class="dropdown-item" @click="ViewProfile">Profile</button>
					    <button class="dropdown-item" @click="doLogout">Logout</button>
				    </div>
			</div>
	</div>




    <div class="d-flex justify-content-center align-items-center flex-column">    
        <div class="input-group" style="max-width: 420px;">
            <input type="file" accept="image/*" class="form-control" @change="uploadFile" ref="file">
            <button class="btn btn-success" style="background-color: #8A3FFC;" @click="submitFile">Upload</button>
        </div>


        <div v-if="successmsg" class="mt-3 alert alert-success">
            {{ successmsg }}
            <router-link to="/users/{{ username }}/profile">Go to your profile</router-link> to checkout your latest WASAUpload
        </div>
    </div>



</template>