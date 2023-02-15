# Octane migration guides

These guides are a lightly tweaked version of the guides I developed for LinkedIn's flagship app's migration to Ember Octane in late 2020. (I removed all the LinkedIn-specific bits, and added a couple new links, but that’s it!) The migration ran in earnest in 2021, with most of the millions of lines of code fully migrated by mid-2021.

- [Phase 1: Native Classes](./Phase%201%20-%20Native%20Classes.md)
- [Phase 2: Improved Templates](./Phase%202%20-%20Improved%20Templates.md)
- [Phase 3: Glimmer Components and `@tracked` Properties](./Phase%203%20-%20Glimmer%20Components%20and%20Tracked%20Properties.md)

The order here is important: doing it with this sequence—and with the detailed sequences described in the per-phase guides!—minimizes the number of times you have to touch each file and maximizes the benefit of each phase of migration.

Additionally, I have included a version of the 1-pager I wrote as part of our process for getting the migration prioritized, [here](./Octane%201-Pager.pdf). As with the guides, it has had LinkedIn-specific/internal-only links removed.
