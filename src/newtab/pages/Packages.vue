<template>
  <div class="container py-8 pb-4">
    <h1 class="text-2xl font-semibold">
      {{ $t('common.packages') }}
    </h1>
    <div class="mt-8 flex items-start">
      <div class="mr-8 hidden w-60 lg:block">
        <div class="flex items-center">
          <ui-button
            class="w-full rounded-r-none border-r"
            variant="accent"
            @click="addState.show = true"
          >
            <p>{{ t('packages.new') }}</p>
          </ui-button>
          <hr />
          <ui-popover>
            <template #trigger>
              <ui-button icon variant="accent" class="rounded-l-none">
                <v-remixicon name="riArrowDropDownLine" />
              </ui-button>
            </template>
            <ui-list>
              <ui-list-item
                v-close-popover
                class="cursor-pointer"
                @click="importPackage"
              >
                {{ t('packages.import') }}
              </ui-list-item>
            </ui-list>
          </ui-popover>
        </div>
        <ui-list class="mt-4 space-y-1 text-gray-600 dark:text-gray-200">
          <ui-list-item
            v-for="cat in categories"
            :key="cat.id"
            :active="cat.id === state.activeCat"
            class="cursor-pointer"
            color="bg-box-transparent text-black dark:text-gray-100"
            @click="state.activeCat = cat.id"
          >
            {{ cat.name }}
          </ui-list-item>
        </ui-list>
      </div>
      <div class="flex-1">
        <div class="flex flex-wrap items-center">
          <div class="flex w-full items-center md:w-auto">
            <ui-input
              v-model="state.query"
              :placeholder="t('common.search')"
              class="flex-1"
              prepend-icon="riSearch2Line"
            />
            <ui-button
              variant="accent"
              class="ml-4 lg:hidden"
              @click="addState.show = true"
            >
              <v-remixicon name="riAddLine" class="mr-2 -ml-1" />
              <span>{{ t('common.packages') }}</span>
            </ui-button>
          </div>
          <div class="grow" />
          <div class="workflow-sort mt-4 flex items-center lg:mt-0">
            <ui-button
              icon
              class="rounded-r-none border-r border-gray-300 dark:border-gray-700"
              @click="
                sortState.order = sortState.order === 'asc' ? 'desc' : 'asc'
              "
            >
              <v-remixicon
                :name="sortState.order === 'asc' ? 'riSortAsc' : 'riSortDesc'"
              />
            </ui-button>
            <ui-select v-model="sortState.by" :placeholder="t('sort.sortBy')">
              <option v-for="sort in sorts" :key="sort" :value="sort">
                {{ t(`sort.${sort}`) }}
              </option>
            </ui-select>
          </div>
        </div>
        <div
          class="mt-8 grid grid-cols-1 gap-4 md:grid-cols-2 lg:grid-cols-3 2xl:grid-cols-4"
        >
          <ui-card
            v-for="pkg in packages"
            :key="pkg.id"
            class="group flex flex-col hover:ring-2 hover:ring-accent dark:hover:ring-gray-200"
          >
            <div class="flex items-center">
              <ui-img
                v-if="pkg.icon?.startsWith('http')"
                :src="pkg.icon"
                class="overflow-hidden rounded-lg"
                style="height: 40px; width: 40px"
                alt="Can not display"
              />
              <span v-else class="bg-box-transparent rounded-lg p-2">
                <v-remixicon :name="pkg.icon || 'mdiPackageVariantClosed'" />
              </span>
              <div class="grow" />
              <ui-popover>
                <template #trigger>
                  <v-remixicon
                    name="riMoreLine"
                    class="cursor-pointer text-gray-600 dark:text-gray-200"
                  />
                </template>
                <ui-list class="space-y-1" style="min-width: 180px">
                  <ui-list-item
                    v-if="pkg.isExternal"
                    v-close-popover
                    :href="`https://extension.automa.site/packages/${pkg.id}`"
                    tag="a"
                    target="_blank"
                    class="cursor-pointer"
                  >
                    <v-remixicon name="riExternalLinkLine" class="mr-2 -ml-1" />
                    <span>Open package page</span>
                  </ui-list-item>
                  <template v-else>
                    <ui-list-item
                      v-close-popover
                      class="cursor-pointer"
                      @click="duplicatePackage(pkg)"
                    >
                      <v-remixicon name="riFileCopyLine" class="mr-2 -ml-1" />
                      <span>{{ t('common.duplicate') }}</span>
                    </ui-list-item>
                    <ui-list-item
                      v-close-popover
                      class="cursor-pointer"
                      @click="exportPackage(pkg)"
                    >
                      <v-remixicon name="riDownloadLine" class="mr-2 -ml-1" />
                      <span>{{ t('common.export') }}</span>
                    </ui-list-item>
                  </template>
                  <ui-list-item
                    v-close-popover
                    class="cursor-pointer text-red-500 dark:text-red-400"
                    @click="deletePackage(pkg)"
                  >
                    <v-remixicon name="riDeleteBin7Line" class="mr-2 -ml-1" />
                    <span>{{ t('common.delete') }}</span>
                  </ui-list-item>
                </ui-list>
              </ui-popover>
            </div>
            <router-link
              :to="`/packages/${pkg.id}`"
              class="mt-4 flex-1 cursor-pointer"
            >
              <p class="text-overflow font-semibold">
                {{ pkg.name }}
              </p>
              <p
                class="line-clamp leading-tight text-gray-600 dark:text-gray-200"
              >
                {{ pkg.description }}
              </p>
            </router-link>
            <div
              class="mt-2 flex items-center text-gray-600 dark:text-gray-200"
            >
              <p>{{ dayjs(pkg.createdAt).fromNow() }}</p>
              <p v-if="pkg.author" class="text-overflow ml-4 flex-1 text-right">
                By {{ pkg.author }}
              </p>
            </div>
          </ui-card>
        </div>
      </div>
    </div>
    <ui-modal
      v-model="addState.show"
      :title="t('packages.add')"
      @close="clearNewPackage"
    >
      <ui-input
        v-model="addState.name"
        :placeholder="t('common.name')"
        autofocus
        class="w-full"
        @keyup.enter="addPackage"
      />
      <ui-textarea
        v-model="addState.description"
        :placeholder="t('common.description')"
        style="min-height: 200px"
        class="mt-2 w-full"
      />
      <div class="mt-6 flex space-x-4">
        <ui-button class="flex-1" @click="clearNewPackage">
          {{ t('common.cancel') }}
        </ui-button>
        <ui-button class="flex-1" variant="accent" @click="addPackage">
          {{ t('packages.add') }}
        </ui-button>
      </div>
    </ui-modal>
  </div>
</template>
<script setup>
import { useDialog } from '@/composable/dialog';
import dayjs from '@/lib/dayjs';
import { usePackageStore } from '@/stores/package';
import dataExporter from '@/utils/dataExporter';
import { arraySorter, openFilePicker, parseJSON } from '@/utils/helper';
import { computed, reactive } from 'vue';
import { useI18n } from 'vue-i18n';

const { t } = useI18n();
const dialog = useDialog();
const packageStore = usePackageStore();

const sorts = ['name', 'createdAt'];
const categories = [
  { id: 'all', name: t('common.all') },
  { id: 'user-pkgs', name: t('packages.categories.my') },
  { id: 'installed-pkgs', name: t('packages.categories.installed') },
];

const state = reactive({
  query: '',
  activeCat: 'all',
});
const sortState = reactive({
  order: 'desc',
  by: 'createdAt',
});
const addState = reactive({
  show: false,
  name: '',
  description: '',
});

const packages = computed(() => {
  const filtered = packageStore.packages.filter((item) => {
    let isInCategory = true;
    const query = item.name
      .toLocaleLowerCase()
      .includes(state.query.toLocaleLowerCase());

    if (state.activeCat !== 'all') {
      isInCategory =
        state.activeCat === 'user-pkgs' ? !item.isExternal : item.isExternal;
    }

    return isInCategory && query;
  });

  return arraySorter({
    data: filtered,
    key: sortState.by,
    order: sortState.order,
  });
});

function duplicatePackage(pkg) {
  const copyPkg = JSON.parse(JSON.stringify(pkg));
  delete copyPkg.id;

  copyPkg.name += ' - copy';

  packageStore.insert(copyPkg);
}
function importPackage() {
  openFilePicker(['application/json']).then(([file]) => {
    if (file.type !== 'application/json') return;

    const fileReader = new FileReader();
    fileReader.onload = () => {
      const pkgJson = parseJSON(fileReader.result, null);
      if (!pkgJson) return;
      if (!pkgJson.name || !pkgJson.data) return;

      packageStore.insert(pkgJson);
    };
    fileReader.readAsText(file);
  });
}
function exportPackage(pkg) {
  const copyPkg = JSON.parse(JSON.stringify(pkg));
  delete copyPkg.id;

  const blobUrl = dataExporter(
    copyPkg,
    { type: 'json', name: `${pkg.name}.automa-pkg` },
    true
  );
  URL.revokeObjectURL(blobUrl);
}
function deletePackage({ id, name }) {
  dialog.confirm({
    title: 'Delete package',
    body: `Are you sure want to delete "${name}" package?`,
    okVariant: 'danger',
    okText: 'Delete',
    onConfirm: () => {
      packageStore.delete(id);
    },
  });
}
function clearNewPackage() {
  Object.assign(addState, {
    name: '',
    show: false,
    description: '',
  });
}
async function addPackage() {
  try {
    await packageStore.insert({
      name: addState.name.trim() || 'Unnamed',
      description: addState.description,
    });

    clearNewPackage();
  } catch (error) {
    console.error(error);
  }
}
</script>
