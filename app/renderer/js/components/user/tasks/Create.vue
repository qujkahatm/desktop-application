<template>
  <el-container class="settings-container">
    <el-form
      ref="task"
      :model="task"
      :rules="rules"
      label-position="top"
    >
      <el-form-item
        :label="$t('Name of a new task')"
        prop="name"
      >
        <el-input
          v-model="task.name"
          :placeholder="$t('Retrospective')"
        />
      </el-form-item>
      <el-form-item
        :label="$t('Description')"
        prop="description"
      >
        <el-input
          v-model="task.description"
          type="textarea"
          :rows="2"
          :placeholder="$t('The 3:30 PM daily retrospective with Engineering team')"
        />
      </el-form-item>
      <el-form-item
        :label="$t('Project')"
        prop="projectId"
        :error="taskSelectorError"
      >
        <el-cascader
          :v-model="task.projectId"
          :placeholder="$t('Internal projects only')"
          :options="projects"
          filterable
          @change="taskSelectorChange"
        />
        &nbsp;&nbsp;
        <el-button
          type="success"
          icon="el-icon-check"
          plain
          :loading="requestInProgress"
          @click="createTask()"
        >
          {{ $t('Create a new task') }}
        </el-button>
      </el-form-item>
    </el-form>
  </el-container>
</template>

<script>
import Message from '../../Message.vue';

export default {
  name: 'CreateTask',
  props: {
    requestInProgress: {
      type: Boolean,
      default: false,
    },
  },
  data() {

    const projectsMap = this.$store.getters.projects
      .filter(project => project.source === 'internal')
      .map(project => ({ value: String(project.id), label: String(project.name) }));

    return {

      projects: projectsMap,
      taskSelectorError: '',
      task: {
        name: '',
        projectId: [],
        description: '',
      },
      rules: {
        name: [{ required: true, message: this.$t('Task name is necessary') }],
      },

    };

  },
  methods: {

    /**
     * Creates a new task
     */
    createTask() {

      this.$refs.task.validate(async () => {

        // Checking if the project is selected
        if (this.task.projectId.length === 0) {

          this.taskSelectorError = this.$t('Project should be selected');
          return false;

        }

        // Reset project selector error
        this.taskSelectorError = '';

        const createdTask = await this.$ipc.request('tasks/create', this.task);
        if (createdTask.code !== 200) {

          return this.$msgbox({
            title: this.$t('Houston, we have a problem'),
            message: this.$createElement(Message, {
              props: {
                title: `Error ${createdTask.body.id}`,
                message: createdTask.body.message,
              },
            }),
            confirmButtonText: this.$t('Ok'),
          });

        }

        const tasks = await this.$ipc.request('tasks/sync', {});
        this.$store.dispatch('syncTasks', tasks.body);
        this.$router.push({ name: 'user.tasks' });

      });

    },

    /**
     * Handle project change event
     * @param {String} val Selected project ID
     */
    taskSelectorChange(val) {

      this.task.projectId = val;

    },

  },
};
</script>
