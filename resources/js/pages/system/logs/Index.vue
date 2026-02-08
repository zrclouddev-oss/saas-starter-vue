<script setup lang="ts">
import { ref, computed } from 'vue';
import AppLayout from '@/layouts/AppLayout.vue';
import { Head, router } from '@inertiajs/vue3';
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card';
import { Badge } from '@/components/ui/badge';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Search, RefreshCw, FileText, AlertTriangle, Info, AlertCircle, Bug } from 'lucide-vue-next';
import { cn } from '@/lib/utils';

interface LogFile {
    name: string;
    date: string | null;
    size: string;
    modified: number;
}

interface LogEntry {
    timestamp: string;
    env: string;
    level: string;
    message: string;
}

interface LogViewerProps {
    files: LogFile[];
    currentFile: { name: string; size: string } | null;
    logs: LogEntry[];
}

const props = defineProps<LogViewerProps>();

const search = ref('');
const filterLevel = ref<string | null>(null);

const filteredLogs = computed(() => {
    return props.logs.filter((log) => {
        const matchesSearch = log.message.toLowerCase().includes(search.value.toLowerCase()) || 
                              log.timestamp.includes(search.value);
        const matchesLevel = filterLevel.value ? log.level === filterLevel.value : true;
        return matchesSearch && matchesLevel;
    });
});

const getLevelColor = (level: string) => {
    switch (level) {
        case 'error':
        case 'critical':
        case 'emergency':
        case 'alert':
            return 'text-red-600 bg-red-50 border-red-200';
        case 'warning':
        case 'notice':
            return 'text-yellow-600 bg-yellow-50 border-yellow-200';
        case 'info':
            return 'text-blue-600 bg-blue-50 border-blue-200';
        case 'debug':
            return 'text-gray-600 bg-gray-50 border-gray-200';
        default:
            return 'text-gray-600 bg-gray-50 border-gray-200';
    }
};

const getLevelIcon = (level: string) => {
    switch (level) {
        case 'error':
        case 'critical':
            return AlertCircle;
        case 'warning':
            return AlertTriangle;
        case 'info':
            return Info;
        case 'debug':
            return Bug;
        default:
            return FileText;
    }
};

const handleFileSelect = (fileName: string) => {
    router.get('/logs', { file: fileName }, { preserveState: true });
};

const handleRefresh = () => {
    router.reload();
};

const toggleFilterLevel = (level: string) => {
    filterLevel.value = filterLevel.value === level ? null : level;
};
</script>

<template>
    <AppLayout
        :breadcrumbs="[
            {
                title: 'Logs',
                href: '/logs',
            },
        ]"
    >
        <Head title="System Logs" />

        <div class="flex h-[calc(100vh-8rem)] gap-6 p-4">
            <!-- Sidebar: File List -->
            <Card class="w-80 flex-shrink-0 flex flex-col">
                <CardHeader class="p-4 border-b">
                    <CardTitle class="text-lg">Log Files</CardTitle>
                    <CardDescription>Select a file to view</CardDescription>
                </CardHeader>
                <CardContent class="p-0 flex-1 overflow-hidden">
                    <div class="h-full overflow-y-auto">
                        <div class="flex flex-col">
                            <button
                                v-for="file in files"
                                :key="file.name"
                                @click="handleFileSelect(file.name)"
                                :class="cn(
                                    'flex items-center justify-between p-3 text-sm text-left hover:bg-muted transition-colors border-b last:border-0',
                                    currentFile?.name === file.name && 'bg-muted font-medium'
                                )"
                            >
                                <div class="flex flex-col truncate">
                                    <span class="truncate">{{ file.name }}</span>
                                    <span class="text-xs text-muted-foreground">{{ file.date || 'Unknown Date' }}</span>
                                </div>
                                <Badge variant="outline" class="ml-2 text-xs">
                                    {{ file.size }}
                                </Badge>
                            </button>
                        </div>
                    </div>
                </CardContent>
            </Card>

            <!-- Main Content: Logs -->
            <Card class="flex-1 flex flex-col overflow-hidden">
                <CardHeader class="p-4 border-b flex flex-row items-center justify-between space-y-0">
                    <div>
                        <CardTitle class="text-lg flex items-center gap-2">
                            {{ currentFile?.name || 'Select a log file' }}
                            <Badge variant="secondary" class="font-mono text-xs">
                                {{ currentFile?.size }}
                            </Badge>
                        </CardTitle>
                        <CardDescription>
                            {{ filteredLogs.length }} entries found
                        </CardDescription>
                    </div>
                    <div class="flex items-center gap-2">
                        <Button 
                            variant="outline" 
                            size="sm" 
                            @click="handleRefresh"
                            title="Refresh Logs"
                        >
                            <RefreshCw class="h-4 w-4" />
                        </Button>
                    </div>
                </CardHeader>
                
                <!-- Filters Toolbar -->
                <div class="p-4 border-b bg-muted/30 flex gap-4 items-center">
                    <div class="relative flex-1 max-w-sm">
                        <Search class="absolute left-2.5 top-2.5 h-4 w-4 text-muted-foreground" />
                        <Input
                            type="text"
                            placeholder="Search logs..."
                            class="pl-9 h-9"
                            v-model="search"
                        />
                    </div>
                    <div class="flex gap-2">
                        <Button
                            v-for="level in ['error', 'warning', 'info', 'debug']"
                            :key="level"
                            :variant="filterLevel === level ? 'default' : 'outline'"
                            size="sm"
                            @click="toggleFilterLevel(level)"
                            :class="cn(
                                'capitalize h-9',
                                filterLevel === level && getLevelColor(level).replace('bg-', 'bg-').replace('text-', '')
                            )"
                        >
                            {{ level }}
                        </Button>
                    </div>
                </div>

                <CardContent class="p-0 flex-1 overflow-hidden bg-slate-950">
                    <div class="h-full overflow-y-auto">
                        <div class="p-4 font-mono text-xs space-y-1">
                            <div v-if="filteredLogs.length === 0" class="text-slate-500 text-center py-10 italic">
                                No log entries found.
                            </div>
                            <div
                                v-else
                                v-for="(log, index) in filteredLogs"
                                :key="index"
                                :class="cn(
                                    'flex gap-3 p-2 rounded hover:bg-slate-900/50 transition-colors border-l-2',
                                    getLevelColor(log.level).replace('text-', 'border-')
                                )"
                            >
                                <div class="text-slate-500 w-32 flex-shrink-0 truncate select-none">
                                    {{ log.timestamp }}
                                </div>
                                <div :class="cn('w-16 flex-shrink-0 uppercase font-bold text-center rounded px-1 text-[10px] leading-5 h-5', getLevelColor(log.level))">
                                    {{ log.level }}
                                </div>
                                <div class="flex-1 text-slate-300 break-all whitespace-pre-wrap">
                                    {{ log.message }}
                                </div>
                            </div>
                        </div>
                    </div>
                </CardContent>
            </Card>
        </div>
    </AppLayout>
</template>
