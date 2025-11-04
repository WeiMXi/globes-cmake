# install data files
install(DIRECTORY BetaBeam DESTINATION share/globes
    PATTERN "*.dat" PATTERN "*.glb"
)

install(DIRECTORY NuFact DESTINATION share/globes
    PATTERN "*.dat" PATTERN "*.glb"
)

install(DIRECTORY Reactor DESTINATION share/globes
    PATTERN "*.dat" PATTERN "*.glb"
)

install(DIRECTORY SuperBeam DESTINATION share/globes
    PATTERN "*.dat" PATTERN "*.glb"
)