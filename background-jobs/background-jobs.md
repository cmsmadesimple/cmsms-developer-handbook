## Background Jobs (CmsJobManager)

CMSMS 2.2 introduced an asynchronous job system that allows modules to schedule background tasks — one-time jobs or recurring cron jobs. Jobs are managed by the core **CmsJobManager** module and executed automatically during page requests.

### How It Works

1. Your module creates a job object (extending `\CMSMS\Async\Job` or `\CMSMS\Async\CronJob`).
2. You call `$job->save()` to add it to the job queue in the database.
3. The CmsJobManager module checks for pending jobs on each page request and executes them when their start time has passed.
4. For cron jobs, after execution the next run time is calculated based on the frequency and the job is re-queued automatically.

> **Note:** The CmsJobManager module must be installed and enabled for background jobs to work. It is included with CMSMS 2.2+ but may need to be activated.

### One-Time Jobs

A one-time job runs once and is removed from the queue after execution. Extend `\CMSMS\Async\Job`:

```php
&lt;?php
// lib/class.SendNotificationJob.php
if (!defined('CMS_VERSION')) exit;

use \CMSMS\Async\Job;

class SendNotificationJob extends Job
{
    private $_email;
    private $_message;

    public function __construct()
    {
        parent::__construct();
        $this->module = 'Holidays';  // Associate with your module
    }

    public function __get($key)
    {
        if ($key === 'email') return $this->_email;
        if ($key === 'message') return $this->_message;
        return parent::__get($key);
    }

    public function __set($key, $val)
    {
        if ($key === 'email') { $this->_email = $val; return; }
        if ($key === 'message') { $this->_message = $val; return; }
        parent::__set($key, $val);
    }

    public function execute()
    {
        $mailer = \cms_utils::get_module('CMSMailer');
        if (!$mailer) return false;

        $mailer->reset();
        $mailer->AddAddress($this->_email);
        $mailer->SetBody($this->_message);
        $mailer->SetSubject('Holiday Notification');
        $result = $mailer->Send();

        return ($result) ? true : false;
    }
}
```

#### Scheduling the job

```
// In an action file or anywhere in your module
$job = new SendNotificationJob();
$job->name = 'holiday_notification_' . $holiday->id;
$job->email = $recipient_email;
$job->message = 'A new holiday has been added: ' . $holiday->name;
$job->start = time(); // Execute as soon as possible
$job->save();
```

### Recurring Cron Jobs

A cron job runs repeatedly at a specified frequency. Extend `\CMSMS\Async\CronJob`:

```php
&lt;?php
// lib/class.CleanupJob.php
if (!defined('CMS_VERSION')) exit;

use \CMSMS\Async\CronJob;

class CleanupJob extends CronJob
{
    public function __construct()
    {
        parent::__construct();
        $this->module = 'Holidays';
        $this->frequency = self::RECUR_DAILY;
    }

    public function execute()
    {
        // Delete holidays older than 2 years
        $cutoff = strtotime('-2 years');
        $db = \cms_utils::get_db();
        $sql = 'DELETE FROM ' . CMS_DB_PREFIX . 'mod_holidays
                WHERE the_date &lt; ? AND published = 0';
        $db->Execute($sql, [$cutoff]);

        audit('', 'Holidays', 'Cleanup job executed');
        return true;
    }
}
```

### Frequency Constants

The `CronJobInterface` defines these frequency constants:

| Constant | Frequency |
| --- | --- |
| `RECUR_NONE` | Does not recur (one-time, but using CronJob class) |
| `RECUR_3M` | Every 3 minutes |
| `RECUR_5M` | Every 5 minutes |
| `RECUR_10M` | Every 10 minutes |
| `RECUR_15M` | Every 15 minutes |
| `RECUR_30M` | Every 30 minutes |
| `RECUR_HOURLY` | Every hour |
| `RECUR_2H` / `RECUR_120M` | Every 2 hours |
| `RECUR_3H` / `RECUR_180M` | Every 3 hours |
| `RECUR_6H` | Every 6 hours |
| `RECUR_12H` | Every 12 hours |
| `RECUR_DAILY` | Once per day |
| `RECUR_WEEKLY` | Once per week |
| `RECUR_MONTHLY` | Once per month |

### Saving and Deleting Jobs

```
// Save a job to the queue
$job = new CleanupJob();
$job->name = 'holidays_daily_cleanup';
$job->save();
// $job->id is now set

// Delete a specific job
$job->delete();

// Delete all jobs from your module (e.g., in method.uninstall.php)
$jobmgr = \CMSMS\Async\JobManager::get_instance();
$jobmgr->delete_jobs_by_module('Holidays');
```

### Job Properties

| Property | Type | Description |
| --- | --- | --- |
| `id` | int (read-only) | Unique ID, assigned on save |
| `name` | string | Job name. Auto-generated if not set. Use a descriptive name for debugging. |
| `module` | string | The module that created this job. Set this so jobs can be cleaned up on uninstall. |
| `start` | int | Unix timestamp — the earliest time the job should execute. Defaults to now. |
| `created` | int (read-only) | Unix timestamp when the job was first created |
| `errors` | int (read-only) | Number of errors encountered during execution attempts |

For cron jobs, additional properties from `CronJobTrait`:

| Property | Type | Description |
| --- | --- | --- |
| `frequency` | string | One of the RECUR\_\* constants |
| `until` | int | Unix timestamp — stop recurring after this date (0 = no end) |

### Real-World Example: Scheduled Backups

Here's a pattern from a real module that schedules backup jobs based on admin settings:

```php
&lt;?php
// lib/class.BackupJob.php
use \CMSMS\Async\CronJob;

class BackupJob extends CronJob
{
    private $_backup_type = 'complete';

    public function __construct()
    {
        parent::__construct();
        $this->module = 'Backups';
        $this->frequency = self::RECUR_NONE;
    }

    public function __get($key)
    {
        if ($key === 'backup_type') return $this->_backup_type;
        return parent::__get($key);
    }

    public function __set($key, $val)
    {
        if ($key === 'backup_type') { $this->_backup_type = $val; return; }
        parent::__set($key, $val);
    }

    public function execute()
    {
        $mod = \cms_utils::get_module('Backups');
        if (!$mod) return false;

        try {
            // Perform the backup based on type
            $backupManager = new BackupManager($mod);
            $result = $backupManager->CreateBackup($this->backup_type);
            audit('', 'CmsJobMgr', 'Backup completed: ' . $this->backup_type);
            return (bool) $result;
        } catch (\Exception $e) {
            audit('', 'CmsJobMgr', 'Backup failed: ' . $e->getMessage());
            return false;
        }
    }
}
```

#### Scheduling from a settings form

```php
// In your settings action — update the scheduled job when settings change
public function UpdateScheduledJob()
{
    // Delete existing jobs for this module
    $jobmgr = \CMSMS\Async\JobManager::get_instance();
    $jobmgr->delete_jobs_by_module($this->GetName());

    // Create new job based on the schedule preference
    $schedule = $this->GetPreference('db_schedule', 'manual');
    if ($schedule !== 'manual') {
        $job = new BackupJob();
        $job->name = 'scheduled_backup';
        $job->frequency = $schedule;  // e.g., '_daily', '_weekly'
        $job->backup_type = 'database';
        $job->save();
    }
}
```

### Custom Data in Jobs

Store custom data by adding private properties with `__get` / `__set` overrides, as shown in the examples above. The job object is serialized when saved to the database, so all private properties are preserved.

> **Note:** Jobs execute without an admin session. You cannot rely on session variables, the current user, or any request-specific data. Store everything the job needs in the job object itself or in the database.

### Cleaning Up on Uninstall

Remove all your module's jobs in `method.uninstall.php`:

```php
// method.uninstall.php
try {
    $jobmgr = \CMSMS\Async\JobManager::get_instance();
    $jobmgr->delete_jobs_by_module($this->GetName());
} catch (\Exception $e) {
    // CmsJobManager may not be available
}
```

### The Legacy PseudoCron System

Before CMSMS 2.2, modules used the `CmsRegularTask` interface and the `get_tasks()` method on the module class for recurring tasks. This older system still works but the async job system is preferred for new development.

The core `\CMSMS\Async\RegularTask` class provides a bridge that converts old `CmsRegularTask` implementations into async jobs automatically.

### API Reference

#### \CMSMS\Async\Job (abstract)

| Method | Description |
| --- | --- |
| `execute()` | Abstract — implement your job logic here. Return `true` on success, `false` on failure. |
| `save()` | Save the job to the database queue. Sets the `id` property. |
| `delete()` | Remove the job from the queue. |

#### \CMSMS\Async\CronJob (abstract, extends Job)

Adds the `frequency` and `until` properties via `CronJobTrait`. After execution, the job is automatically rescheduled based on the frequency.

#### \CMSMS\Async\JobManager (singleton)

| Method | Description |
| --- | --- |
| `get_instance()` | Get the singleton instance |
| `load_job($id)` | Load a job by its integer ID |
| `save_job($job)` | Save a job to the queue (returns the job ID) |
| `delete_job($job)` | Remove a job from the queue |
| `delete_jobs_by_module($name)` | Remove all jobs created by a specific module |
