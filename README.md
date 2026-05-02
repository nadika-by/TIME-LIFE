// [EN] Smart contract logic for Time & Life ecosystem
// [RU] Логика смарт-контракта для экосистемы Time & Life

use anchor_lang::prelude::*;

declare_id!("TimeLifeContract1111111111111111111111111");

#[program]
pub mod time_and_life {
    use super::*;

    // Инициализация профиля гражданина (привязка к биометрии)
    pub fn create_user_profile(ctx: Context<CreateProfile>, fingerprint_hash: [u8; 32]) -> Result<()> {
        let profile = &mut ctx.accounts.user_profile;
        profile.authority = *ctx.accounts.signer.key;
        profile.fingerprint_hash = fingerprint_hash;
        profile.health_index = 100; // Стартовый индекс здоровья
        profile.labor_points = 0;   // Баллы трудовой лояльности
        profile.is_active = true;
        Ok(())
    }

    // Метод начисления TimeCoin на основе налоговых отчислений (ОСМС/ОПВР)
    pub fn claim_tax_reward(ctx: Context<UpdateProfile>, amount: u64) -> Result<()> {
        let profile = &mut ctx.accounts.user_profile;
        // Только если индекс здоровья выше 80, разрешаем кэшбэк
        if profile.health_index >= 80 {
            profile.labor_points += amount;
        }
        Ok(())
    }

    // Фиксация материального ущерба (удержание из накопленного капитала)
    pub fn report_damage(ctx: Context<UpdateProfile>, damage_cost: u64) -> Result<()> {
        let profile = &mut ctx.accounts.user_profile;
        if profile.labor_points >= damage_cost {
            profile.labor_points -= damage_cost;
        } else {
            profile.labor_points = 0;
        }
        Ok(())
    }
}

#[account]
pub struct UserProfile {
    pub authority: Pubkey,       // Владелец кошелька
    pub fingerprint_hash: [u8; 32], // Хеш отпечатка пальца
    pub health_index: u8,        // Индекс здоровья (ЗОЖ)
    pub labor_points: u64,       // Накопленные TimeCoin (из налогов)
    pub is_active: bool,
}

#[derive(Accounts)]
pub struct CreateProfile<'info> {
    #[account(init, payer = signer, space = 8 + 32 + 32 + 1 + 8 + 1)]
    pub user_profile: Account<'info, UserProfile>,
    #[account(mut)]
    pub signer: Signer<'info>,
    pub system_program: Program<'info, System>,
}

#[derive(Accounts)]
pub struct UpdateProfile<'info> {
    #[account(mut, has_one = authority)]
    pub user_profile: Account<'info, UserProfile>,
    pub authority: Signer<'info>,
}
