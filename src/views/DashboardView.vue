<template>
    <div class="dashboard-view">
        <div class="dashboard-container">
            <div class="header-row">
                <div class="user-info-card">
                    <img :src="$store.state.avatar_url" alt="User Avatar" class="avatar">
                    <h2>Welcome, {{ $store.state.user.username }}!</h2>
                    <p>Here's a quick overview of your health and activities.</p>
                </div>

                <div class="health-overview">
                    <div class="card">
                        <h3>Calories Consumed Today</h3>
                        <p v-if="userCaloriesConsumed">{{ userCaloriesConsumed }} kcal</p>
                        <p v-else>No data available</p>
                    </div>
                    <div class="card">
                        <h3>Calories Burned Today</h3>
                        <p v-if="userCaloriesBurned">{{ userCaloriesBurned }} kcal</p>
                        <p v-else>No data available</p>
                    </div>
                </div>
            </div>
            <div class="main-content">
                <div class="recent-diet">
                    <h2>Recent Diet</h2>
                    <div class="log-cards">
                        <div v-for="diet in recentDiets" :key="diet.id" class="log-card">
                            <div class="log-content">
                                <div class="log-header">
                                    <h3>{{ diet.food_name }}</h3>
                                    <p>{{ diet.quantity }} grams comsumed {{ diet.calories_consumed }} kcal</p>
                                </div>
                                <footer class="log-footer">
                                    <small>Logged on {{ formatDate(diet.date) }}</small>
                                </footer>
                            </div>
                        </div>
                    </div>
                    <div v-if="!recentDiets.length" class="no-logs">
                        <p>No diet logs available. Start by adding your first log!</p>
                    </div>
                </div>
                <div class="recent-activity">
                    <h2>Recent Activity</h2>
                    <div class="log-cards">
                        <div v-for="activity in recentActivities" :key="activity.id" class="log-card">
                            <div class="log-content">
                                <div class="log-header">
                                    <h3>{{ activity.activity_type }}</h3>
                                    <p>{{ activity.duration }} minutes burned {{ activity.calories_burned }} kcal</p>
                                </div>
                                <footer class="log-footer">
                                    <small>Logged on {{ formatDate(activity.date) }}</small>
                                </footer>
                            </div>
                        </div>
                    </div>
                    <div v-if="!recentActivities.length" class="no-logs">
                        <p>No sport logs available. Start by adding your first log!</p>
                    </div>
                </div>
                <div class="community-feed">
                    <h2>Community Feed</h2>
                    <div class="post-cards">
                        <div v-for="post in posts" :key="post.id" class="post-card"
                            @click="navigateTo(`/community/post?id=${post.id}`)">
                            <div v-if="post.image_url" class="post-image">
                                <img :src="`${$http.defaults.baseURL}${post.image_url}`" alt="Post Cover" />
                            </div>
                            <div class="post-content">
                                <div class="post-header">
                                    <h3>{{ post.title }}</h3>
                                    <p>{{ post.summary }}</p>
                                </div>
                                <footer class="post-footer">
                                    <div class="author-avatar">
                                        <img :src="`${$http.defaults.baseURL}${post.author_avatar_url}`"
                                            alt="Author Avatar" />
                                    </div>
                                    <small>{{ post.author }} | {{ formatDate(post.date) }}</small>
                                </footer>
                            </div>
                        </div>
                        <div v-if="!posts.length" class="no-posts">No posts available.</div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>

export default {
    name: 'DashboardView',
    data() {
        return {
            userCaloriesBurned: null,
            userCaloriesConsumed: null,
            recentDiets: [],
            recentActivities: [],
            posts: []
        };
    },
    mounted() {
        this.fetchData();
    },
    methods: {
        navigateTo(route) {
            this.$router.push(route);
        },
        fetchData() {
            const token = localStorage.getItem('jwtToken');
            this.$http.get('/api/dashboard/fetch', { headers: { Authorization: `Bearer ${token}` } })
                .then(response => {
                    this.userCaloriesBurned = response.data.userCaloriesBurned;
                    this.userCaloriesConsumed = response.data.userCaloriesConsumed;
                    this.recentDiets = response.data.recentDiets;
                    this.recentActivities = response.data.recentActivities;
                    this.posts = response.data.posts;
                })
                .catch(error => console.log(error));
        },
        formatDate(dateString) {
            const date = new Date(dateString);
            return date.toLocaleDateString();
        }
    }
};
</script>

<style scoped>
@import url(../assets/post-cards.css);
@import url(../assets/log-cards.css);

.dashboard-view {
    display: flex;
    flex-direction: column;
    justify-content: center;
    width: 100%;
    height: 98vh;
    background-color: #eeeeee;
    gap: 0;
}

.dashboard-container {
    width: 90%;
    height: 85%;
    margin: 0 auto;
    margin-top: 60px;
    padding: 20px;
    background-color: white;
    border-radius: 8px;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

.header-row {
    display: flex;
    justify-content: space-between;
    align-items: flex-start;
    height: 30%;
    margin-bottom: 20px;
}

.user-info-card {
    text-align: center;
    flex: 1;
    margin-right: 20px;
}

.user-info-card .avatar {
    width: 100px;
    height: 100px;
    border-radius: 50%;
    margin-bottom: 10px;
}

.health-overview {
    display: flex;
    justify-content: space-between;
    flex: 1;
}

.health-overview .card {
    padding: 20px;
    border-radius: 8px;
    text-align: center;
    flex: 1;
}

.main-content {
    flex: 1;
    overflow-y: auto;
    height: 60%;
    padding: 20px;
    border-top: 1px solid #ccc;
    display: flex;
    flex-direction: column;
    align-items: center;
}

.recent-diet,
.recent-activity,
.community-feed {
    width: 90%;
    max-width: 800px;
}

.recent-diet h2,
.recent-activity h2,
.community-feed h2 {
    font-size: 18px;
    margin-bottom: 10px;
    color: #45a049;
}
</style>